# For update:

New version;

```
# Fix the file first
sed -i 's/\r//' smash

# Create new tarball
cd ~/smash
tar czf smash_1.2-1.orig.tar.gz --exclude=smash-1.2/debian smash-1.2-1/

# Bump version
cd smash-1.2-1
#dch -v 1.2-1 "Fix float typecasting and clean output improvements"

# Build and upload
debuild -S -sa -kYOURKEY
dput ppa:flaneurette/smash smash_1.2-1_source.changes
```

# Update:

```
# Fix line endings
sed -i 's/\r//' smash

# Verify
file smash
# should say: ASCII text executable

# Create new tarball
tar czf ../smash_1.0.orig.tar.gz smash-1.0/

# Bump version
dch -v 1.0-1 "Fix Windows line endings"

# Build and upload
debuild -S -sa -kYOURKEY
dput ppa:flaneurette/smash smash_1.0-1_source.changes
```

```
#### For jammy
dch -v 1.0-0~jammy "Package for Ubuntu 22.04"
#### change noble to jammy in changelog
debuild -S -sa -kYOURKEY
dput ppa:flaneurette/smash smash_1.0-0~jammy_source.changes

#### For focal
dch -v 1.0-0~focal "Package for Ubuntu 20.04"
debuild -S -sa -kYOURKEY
dput ppa:flaneurette/smash smash_1.0-0~focal_source.changes
```

```
// PPA Release Builder .smash script
let package = "smash";
let version = "1.0-0";
let key = "B449A805B697AD1D";
let releases = ["noble", "jammy", "focal"];

for (let release of releases) {
    echo "Building for " + release + "...";
    $(dch -v ${version}~${release} "Package for ${release}");
    $(sed -i "s/noble/${release}/" debian/changelog);
    $(debuild -S -sa -k${key});
    $(dput ppa:flaneurette/${package} ${package}_${version}~${release}_source.changes);
}

echo "All releases uploaded!";

```
#### Update your files in the main folder

#### Then create new tarball
```
tar czf smash_1.1.orig.tar.gz smash-1.1/
```
#### Bump version in debian/changelog
```
dch -v 1.1-0 "Added new features"
```
#### Rebuild and upload
```
debuild -S -sa -kB449A805B697AD1DACC829D2E58EC1824E443681
dput ppa:flaneurette/smash smash_1.1-0_source.change
```
|
|
|
|

# Packaging, Signing, and Uploading a .deb Package

This guide uses the **lcd** tool as an example.

`su - flaneurette`

`cd sources`

## 1. Prepare your Debian package

Directory structure:

```
lcd-1.1/
├── lcd          program
├── lcd.1        man page
└── debian/
```

New:

```
lcd-1.2/
├── lcd
├── lcd.1
├── modes/
│   ├── logs.sh
│   ├── certs.sh
│   └── services.sh
└── debian/
```

Install:
```
lcd              usr/bin/
lcd.1            usr/share/man/man1/
modes/*          usr/share/lcd/modes/
debian/lcd.conf  etc/lcd/
```

Test:

```
dpkg-buildpackage -us -uc
sudo dpkg -i ../lcd_1.1_all.deb
lcd --help
man lcd
```

Then:

```
lintian ../lcd_1.1_all.deb
```

### Create the orig tarball

From `~/sources`:

```sh
tar --exclude-vcs --exclude=debian -czf lcd_1.1.orig.tar.gz lcd-1.1
```

Build the `.deb` locally:

```bash
dpkg-deb --build lcd-1.0
```

Or, for source packages:

```bash
debuild -S -sa -kB449A805B697AD1DACC829D2E58EC1824E443681
```

> `-k<fingerprint>` ensures your **GPG key** is used to sign the source package.

---

## Upload to your PPA

```bash
dput ppa:flaneurette/lcd lcd_1.1-1_source.changes
```

Check that your signature is valid and the upload completes.

---

## Install from PPA (once built by Launchpad)

```bash
sudo add-apt-repository ppa:flaneurette/lcd
sudo apt update
sudo apt install lcd
```

---

Tips:

* Always use the **full GPG fingerprint** to avoid short key issues.
* Ensure your `DEBIAN/control` file and directory structure are correct.
* Launchpad may take a few minutes to build the `.deb` from your source package.

---

## Sign the package (if using `debsign` separately)

```bash
debsign -kB449A805B697AD1DACC829D2E58EC1824E443681 lcd_1.1-1_source.changes
```

This creates a **signed `.changes` file** for Launchpad.

---

## When changing source code.

Assume:

* upstream name: `lcd`
* upstream version: `1.0`
* source directory: `lcd-1.0/`
* **no `debian/` directory inside** (important)

### Make sure the source tree is clean

From **outside** the directory:

```sh
cd ~/sources
```

Inside `lcd-1.0/`, you should have:

* source code
* README, LICENSE, etc.
* no `debian/`
* no build artifacts

If `debian/` exists, move it out temporarily:

```sh
mv lcd-1.0/debian .
```

---

### Create the orig tarball

From `~/sources`:

```sh
tar --exclude-vcs -czf lcd_1.0.orig.tar.gz lcd-1.0
```

That’s it.

This produces exactly what Debian expects:

```
lcd_1.0.orig.tar.gz
```

---

### Put `debian/` back

```sh
mv debian lcd-1.0/
```

Now you have:

* pristine upstream tarball
* Debian packaging layered on top

---

## Why `debian/` must not be inside `orig.tar.gz`

Debian’s model is:

> upstream source + Debian changes = Debian package

If `debian/` is in the orig tarball:

* diffs get weird
* tooling complains
* reviewers frown politely but firmly

Keeping them separate is the whole point.

---

## Quick sanity check

You can inspect the tarball:

```sh
tar tzf lcd_1.0.orig.tar.gz | head
```

You should **not** see:

```
lcd-1.0/debian/
```

If you do - redo it.

---

## Alternative: let `dpkg-source` do it (optional)

If you’re already set up and lazy (valid):

```sh
dpkg-source -b .
```

This works **only if**:

* source version matches
* orig tarball doesn’t exist yet
* directory layout is correct

I still recommend knowing the manual method - it’s transparent and portable.

---

## Naming rules (burn these into muscle memory)

* Directory: `lcd-1.0/`
* Tarball: `lcd_1.0.orig.tar.gz`
* Version must match **exactly**

Hyphen in dir name → underscore in tarball name.
Always.

---

## Mental model (forever useful)

Think of `orig.tar.gz` as:

> “What I would give someone if they asked for the raw source.”

No distro stuff. No patches. Just the code.

---
