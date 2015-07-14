#GPG from the command line

This is a guide to key management for [GPG](https://gnupg.org/) on the
command line. By no means is this the only way to do it! There's a user
friendly plugin called [Enigmail](https://enigmail.net/home/index.php) for
the free [Thunderbird](https://www.mozilla.org/en-US/thunderbird/) email
client (as well as other clients like Apple Mail) which simplifies a lot
of this key management.

However, understanding the command line aspect of key management can make
some confusing aspects of the whole process more clear (I think!). Anyway,
let's dive in!

##Make a key

We can run:

`gpg --gen-key` 

from the command line to generate a key. However, this requires us to
manually pick a bunch of options which are probably important for the
security of the key, and are kind of overwhelming. If you want to do this
check out this guide:

[Debian guide](http://keyring.debian.org/creating-key.html)

If not we'll use the wizard in Thunderbird/Enigmail! There's a guide to
doing so from the [FSF here](https://emailselfdefense.fsf.org/en/) and
from the [EFF here](https://ssd.eff.org/en) (look under 'how to use GPG
for Linux/Windows/Mac OS').

###Key Revocation

At some point during the Enigmail wizard it should ask you about generating
a key revocation certificate. This is a *really good idea* and you should do
it.

How does it work? Well basically we can think of a key as an append only log,
where we can add information (signatures, new user ids, a photo, etc) but once
the info has been pushed up to a keyserver we cannot delete it. 

This seems bad: what if I lose my laptop or it's stolen? I can't delete my key?
Well no, and for good reason: the keyserver would have no way of verifying you
without your private key. Enter revocation certs: this is a little file which
can be used to revoke your public key, which essentially means that a flag will
be appended to the key that says 'hey, don't trust this!' so that folks who try
to send you a message will be notified. Then when you get your new laptop you
can generate a new key and happily encrypt away! Nice!

If you store your revocation cert on the same machine as your private key, then
it's just as vulnerable to the vagaries of laptop life (coffee spills, theft,
rain, etc) as your private key, and that doesn't really help you. So put it
somewhere else! I think if you trust your cloud provider that's not a bad spot,
or on another computer that stays at home, or on flash storage injected into
your forearm (??!?!?!).

If you lose your cert but you still have access to your private key you can
always generate a new revocation cert (provided you know your key's password).

###Expiration date

Key expiration date is another mechanism for key inactivation, which is sort of
a failsafe. Some bullet points:

- A key's expiration date can always be changed, even after it has passed (this
  will unexpire the key)
- A key's expiration date may only be changed if you hold the private key
- An expired key is considered to be revoked, in the same manner as if
  a revocation certification had been appended to it.

This gives us some extra security! If we only have the expiration date a year
or so in the future, it will be less bad (but still bad!) if we lose access to
the private key AND to the revocation certificate. In the meantime you would
probably have to explain to folks why there are two keys bearing your identity,
and there's a (remote) possibility that someone with access to your stolen or
lost laptop could impersonate you (but only to folks you haven't told to mark
your old key as untrustworthy).

This brings me to the point: by default Enigmail sets the expiration time to
something like 2020. This is bad! Change it!

Do

`gpg --edit-key yourkeyid`

and then type `expire`. It will then ask you for an expression like `<n>y`,
which will set how long it will take the key to expire in years. Like
I mentioned above, the expiration date can always be extended if you reach that
date and still control your key. So don't worry about it! Set it for a year or
so in the future.

After you hit enter in the edit shell you'll be prompted for your key password,
and then you can type `save` to save your changes.

##Import a key

At the command line do:

`gpg --import-key /path/to/someoneskey.asc`

GPG implements 'trust' for the public keys stored locally on your machine.
The program will not allow you to encrypt a message to a public key which
you haven't marked as 'trusted' (yes this is tedious). To 'trust' a key:

`gpg --edit 'keyid'`

where 'keyid' is the key you want to 'trust'. This opens the interactive
edit shell. Type `trust`, and then hit `4`, and then type `save`.

My understanding is that although there are different trust levels its
actually a binary state (either it will let you send mail to that public
key or no). 

Dealing with 'trust' is one of the things that the Enigmail plugin
automates for us.

##Fetch keys

We can also fetch keys from the intarwebs using GPG:

`gpg --fetch-keys 'url'`

(e.g. after finding someones public key on a keyserver). It's always
a good idea to verify the fingerprint after doing this!

###What is a keyserver, anyhow?

A keyserver is a server that holds public keys. As I understand it
there are a network of them, and they exchange information/updates so that
each keyserver has a copy of the whole PGP key history (sort of like the
bitcoin blockchain?). 

The idea is that an individual server can go down and the info will still
be available. The keyservers are a crucial component of making GPG work,
they are what enable the web of trust and enable easy exchange of keys.

The address [pool.sks-keyservers.net](pool.sks-keyservers.net) is
generally what you should default to using. It lets you use one address to
connect to the whole pool of keyservers, so you don't need to worry about
a particular one being down. 

Sometimes that address seems not to work, however, and in that case
I generally fall back to [pgp.mit.edu](pgp.mit.edu). 

You can set a default keyserver in `~/.gnupg/gpg.conf` like this:

    keyserver hkp://pool.sks-keyservers.net

If you use most Linux distros you can probably substitute `hkps`
for `hkp`, which is analogous to http/https.

##Export a key to a file

We can export a key to a local file:

`gpg --export -a 'key id' > mykey.asc`

The `-a` option is the same as `--armor`, it puts the key in 'ASCII armor'
that the GPG tool will recognize and import (in the same manner that we
did above). This is handy if you want to sneakernet or email your public
key directly to someone.

##List keys

This will list all keys in your keyring:

`gpg --list-keys`

This will give you info about a particular one:

`gpg --list-key 'keyid'`

##List fingerprints

This will list the fingerprint of all keys in your keyring:

`gpg --fingerprint`

You can also get just one:

`gpg --fingerprint 'key id'`

The fingerprint is normally what you share with other folks, if they have
your email address they can search the keyservers for your key and verify
that the fingerprint matches.

Some people use the 'short fingerprint', which is the last 8 of the full
fingerprint, but I'm told this is not as collision resistant as was once
thought, and is considered bad practice.

##Refresh keys from keyserver

This will contact the keyserver and get new versions of all the public
keys that have changed. 

`gpg --refresh`

I'm not sure what the request that is sent up
looks like, so there may be social network information that is leaked
here.

##Send signatures to the keyserver

This will upload a public key to the keyserver.

`gpg --send-keys 'key_id'`

(e.g. after signing a public key)

Why do we have to specify a particular key? Don't we want to send all our
signatures up? Well, not really. Uploading all your signatures opens you
up to social network analysis, and for this reason GPG defaults to keeping
signatures on the local machine (or, uploading signatures to the WOT is
opt-in).

##Search for a key

We can search for a key from the command line:

`gpg --search-keys 'key id'`

I find this kind of unreliable though? I'm not sure why. Generally I have
better luck just going to (pool.sks-keyservers.net) in the browser. Then
once you have found someone's key you can `gpg --fetch-keys 'url'`.

##Sign a key

So that WOT thing, how does it work? Well, basically the idea is that we
sign other folks public keys when we've verified that the key is not under
the control of an attacker/malicious agent. Then the idea is that we have
some degree of separation based web of trust, where if I trust, say, Bob's
key, and I see that he has signed Carol's key, I can trust to a certain
extent that Carol is a real person. That's the idea, anyway. In practice,
key verification is tedious and time consuming, and relies on humans'
ability to compare strings (not great).

If we want to sign a key we can do:

`gpg --sign-key 'key id'`

It will ask you to confirm and enter your private key password.

##Encrypting files!

OK, so all that is out of the way. How do we encrypt a file? Well, if
you're using an email client like Thunderbird it should basically handle
that for you, for encrypting email.

But GPG can do a lot more than just email! If your use case is different
(e.g. you're encrypting a super secret file on your own computer) you can
do:

`gpg --encrypt myfile`

Then you'll get prompted to fill in the recipient - if it's just for your
own use this will be you. This will write a file called `myfile.gpg`. You
can also pass in data to `stdin`:

`echo 'super secret info' | gpg --encrypt --output encrypted.gpg`

You can also encrypt documents that you'll exchange with someone over
channels other than email, provided that you have their public key. You
just need to specify the correct recipient when you're encrypting.

Alternative if you do all your emailing from the command line this is
probably how you encrypt email too? 

##Decrypting files!

This is pretty straightforward!

`gpg --decrypt myfile.gpg`

That works pretty much how you think it will, it prints the decrypted
contents of the file to `stdout`, along with a little bit of info about
the key used for encryption and so on.

You can suppress that info with the `--quiet` option, which is actually
super handy! Say you need to log in to something from the command line and
it can take a password on `stdin`. You could do:

    echo 'mysecurepassword' | gpg --encrypt --output mypassword.gpg
    gpg --decrypt --quiet mypassword.gpg | myterminalutility

Whoa cool! This is actually what I use to store the passwords for my mail
accounts on my machine, it works pretty smoothly.
