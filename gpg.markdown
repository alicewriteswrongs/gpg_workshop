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
manually pick a bunch of options with are probably important for the
security of the key. If you want to do this check out this guide:

[Debian guide](http://keyring.debian.org/creating-key.html)

If not we'll use the wizard in Thunderbird/Enigmail!

##Import a key

At the command line do:

`gpg --import-key /path/to/someoneskey.asc`

##Export a key to a file

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

`gpg --refresh`

##Send signatures to the keyserver

`gpg --send-keys 'key_id'`

(e.g. after signing a public key)

##Fetch keys

`gpg --fetch-keys 'url'`

(e.g. after finding someones public key on a keyserver)

##Search for a key

`gpg --search-keys 'key id'`

##Sign a key

`gpg --sign-key 'key id'`
