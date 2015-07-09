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

We can also fetch keys from the intarwebs usings gpg:

`gpg --fetch-keys 'url'`

(e.g. after finding someones public key on a keyserver). It's always
a good idea to verify the fingerprint after doing this!

###What is a keyserver, anyhow?

A keyserver is a webserver that holds public keys. As I understand it
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

If you use most modern linux distros you can probably substitute `hkps`
for `hkp`, which is anologous to http/https.

##Export a key to a file

We can export a key to a local file:

`gpg --export -a 'key id' > mykey.asc`

The `-a` option is the same as `--armor`, it puts the key in 'ASCII armor'
that the gpg tool will recognize and import (in the same manner that we
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
up to social network analysis, and for this reason gpg defaults to keeping
signatures on the local machine (or, uploading sigs to the WOT is opt-in).

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
some degrees of separation based web of trust, where if I trust say Bob's
key, and I see that he has signed Carol's key, I can trust to a certain
extent that Carol is a real person. That's the idea, anyway. In practice,
key verification is tedious and time consuming, and relies on humans'
ability to compare strings (not great).

If we want to sign a key we can do:

`gpg --sign-key 'key id'`

It will ask you to confirm and enter your private key password.

##Encrypting files!

Ok, so all that is out of the way. How do we encrypt a file? Well, if
you're using an email client like Thunderbird it should basically handle
that for you.

If your use case is different (e.g. you're encrypting a super secret file
on your own computer) you can do:

`gpg --encrypt myfile`

Then you'll get prompted to fill in the recipient - if it's just for your
own use this will be you. This will write a file called `myfile.gpg`. You
can also pass in data to `stdin`:

`echo 'super secret info' | gpg --encrypt --output encrypted.gpg`

##Decrypting files!

This is pretty straightforward!

`gpg --decrypt myfile.gpg`

That works pretty much how you think it will, it prints the decrypted
contents of the file to `stdout`, along with a little bit of info about
the key used for encryption and so on.

You can suppress that info with the `--quiet` option, which is actually
super handy! Say you need to log in to something from the command line and
it can take a passwords on `stdin`. You could do:

    echo 'mysecurepassword' | gpg --encrypt --output mypassword.gpg
    gpg --decrypt --quiet mypassword.gpg | myterminalutility

Whoa cool! This is actually what I use to store the passwords for my mail
accounts on my machine, it works pretty smoothly.
