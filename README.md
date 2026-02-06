# Packaging, Signing, and Uploading a .deb Package

This guide uses the **Located** tool as an example.

## 1. Prepare your Debian package

Directory structure:

```
located-1.0/
├── debian/
│   └── control (file)
├── usr/
│   └── local/
│       └── bin/
│           └── located (file)
├── usr/
│   └── share/
│       └── man/
│           └── man1/
│               └── located.1 (file)
```

Build the `.deb` locally:

```bash
dpkg-deb --build located-1.0
```

Or, for source packages:

```bash
debuild -S -sa -kB449A805B697AD1DACC829D2E58EC1824E443681
```

> `-k<fingerprint>` ensures your **GPG key** is used to sign the source package.

---

## 2. Sign the package (if using `debsign` separately)

```bash
debsign -kB449A805B697AD1DACC829D2E58EC1824E443681 located_1.0-1_source.changes
```

This creates a **signed `.changes` file** for Launchpad.

---

## 3. Upload to your PPA

```bash
dput ppa:flaneurette/located located_1.0-1_source.changes
```

Check that your signature is valid and the upload completes.

---

## 4. Install from PPA (once built by Launchpad)

```bash
sudo add-apt-repository ppa:flaneurette/located
sudo apt update
sudo apt install located
```

---

Tips:

* Always use the **full GPG fingerprint** to avoid short key issues.
* Ensure your `DEBIAN/control` file and directory structure are correct.
* Launchpad may take a few minutes to build the `.deb` from your source package.
