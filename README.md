# For update:

```
tar --exclude-vcs --exclude=debian -czf smash_1.3.orig.tar.gz smash-1.3
cd smash-1.3
debuild -S -sa -kEE274F961CF4A8143BC68EEFB240191A0B2B7B8C
cd ../
dput ppa:flaneurette/smash smash_1.3-0_source.changes

or:

dput -c ~/.dput.cf  smash_1.3-0_source.changes

```

Generate a new GPG key:

```
gpg --full-generate-key
```

List your keys to find the new key ID:

```
gpg --list-secret-keys --keyid-format LONG
```

Export the new public key:

```
gpg --export --armor B240191A0B2B7B8C > public_key.asc
```

Decrypt Launchpad PGP message:

```
gpg --decrypt --output decrypted_file.txt encrypted_message.asc
```
