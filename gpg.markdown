#GPG related things

First off:
    - what is a public/private keypair?
    - where do they live?
        - keyservers
        - hkp/hkps
        - [pool keyserver](pool.sks-keyservers.net)

##Make a key

We can run:

`gpg --gen-key` 

from the command line to generate a key. However, this requires us to
manually pick a bunch of options which are probably important for the
security of the key, and are kind of overwhelming. If you want to do this
check out this guide:

[Debian guide](http://keyring.debian.org/creating-key.html)

If not we'll use the wizard in Thunderbird/Enigmail!

##Import a key

At the command line do:

`gpg --import-key /path/to/someoneskey.asc`

##Fetch keys

We can also fetch keys from the intarwebs usings gpg:

`gpg --fetch-keys 'url'`

(e.g. after finding someones public key on a keyserver). It's always
a good idea to verify the fingerprint after doing this!


##Export a key to a file

We can export a key to a local file:

`gpg --export -a 'key id' > mykey.asc`

The `-a` option is the same as `--armor`, it puts the key in 'ASCII armor'
that the gpg tool will recognize and import. This is handy if you want to
sneakernet or email your public key directly to someone.

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

Ok, so all that is out of the way. How do we encrypt a file?
