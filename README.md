Simple steps for package creation, signing and uploading to ppa.launchpad.net.

*Remember* to change your public key fingerprints, and shorthand (sub) key.

Mine are for demo purposes:

- ABCDEFGHIJKLMNOPQRSTUVWXYZ
- KLMNOPQRSTUVWXYZ

# For update:

nano ~/.dput.cf

```
[upload]
fqdn = ppa.launchpad.net
method = sftp
incoming = ~flaneurette/ubuntu/%(upload)s
login = flaneurette
```

Then: 

```
tar --exclude-vcs --exclude=debian -czf smash_1.3.orig.tar.gz smash-1.3
cd smash-1.3
debuild -S -sa -kABCDEFGHIJKLMNOPQRSTUVWXYZ
cd ../
dput -c upload:smash smash_1.3-0_source.changes
```

### Generate a new GPG key to decrypt launchpad/ubuntu messages:

```
gpg --full-generate-key
```

List your keys to find the new key ID:

```
gpg --list-secret-keys --keyid-format LONG
```

Export the new public key:

```
gpg --export --armor KLMNOPQRSTUVWXYZ > public_key.asc
```

Decrypt Launchpad PGP message:

```
gpg --decrypt --output decrypted_file.txt encrypted_message.asc
```


### Generate launchpad SSH key (for upload, not signing/decrypting PGP/GPG):

```
sudo apt-get install openssh-client

ssh-keygen -a 100 -t ed25519

Public key:
nano /home/<user>/.ssh/id_ed25519.pub

Private key:
cd /home/<user>/.ssh/
nano id_ed25519
```