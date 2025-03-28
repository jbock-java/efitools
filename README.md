# Motivation for this repo

* [efitools was removed from Fedora 41](https://discussion.fedoraproject.org/t/f41-secure-boot-with-only-your-own-keys/138120) due to build problems.
* [efitools upstream](https://web.git.kernel.org/pub/scm/linux/kernel/git/jejb/efitools.git/) looks unmaintained, last tag is `1.9.2`.
* There's this [arch package](https://gitlab.archlinux.org/archlinux/packaging/packages/efitools), building tag `1.9.2` it's not a source repo though.
* There should be a functioning source repo for efitools.

This repo was created at upstream tag `1.9.2`.

# `TODO`

* github workflow

# Prerequisites

```
sudo dnf group install c-development
sudo dnf install efivar-devel gnu-efi-devel sbsigntools perl-File-Slurp openssl-devel openssl-devel-engine help2man
```

# Building the binaries

```
make
rm -rf build && mkdir build
make DESTDIR=build install
```

This should create some binaries like `build/usr/bin/efi-updatevar`.

# Old `README`:

```
How to use these files

simply typing make will build you everything including sample certificates for
PK, KEK and db.

The prerequisites are the standard development environment, gnu-efi version
3.0q or later, help2man and sbsigntools.

There will be one file called LockDown.efi.  If run on your efi platform in
Setup Mode, this binary will *replace* all the values in the PK, KEK and db
variables with the ones you just generated and place the platform back into
User Mode (booting securely).  If you don't want to replace all the variables,
take a dump of your current variables, see sig-list-to-cert(1), and add them
to the EFI signature list files before creating LockDown.efi

Say you want to concatenate an existing platform-db.esl file, do this:

make DB.esl
cat platform.esl DB.esl > newDB.esl
mv newDB.esl DB.esl

and then make LockDown.efi in the usual way.

All of the EFI programs are also generated in signed form (signed by both db
and KEK).


Loader.efi
==========

This EFI binary is created to boot an unsigned EFI file on the platform. Since
this explicitly breaks the security of the platform, it will first check to
see if the boot binary is naturally executable and execute it if it is (either
it's properly signed or the platform isn't in Secure Boot mode).  If the
binary gives an EFI_ACCESS_DENIED error meaning it isn't properly signed,
Loader.efi will request present user authorisation before proceeding to boot.

The idea is that Loader.efi may serve as a chain for elilo.efi or another boot
loader on distributed linux live and install CDs and even as the boot loader
for the distribution on the hard disk assuming the user does not wish to take
control of the platform and replace the keys.

To build a secure bootable CD, simply use Loader.efi as the usual
/efi/boot/bootX64.efi and place the usual loader in the same directory as the
file boot.efi.

In order to add further convenience, if the user places the platform in setup
mode and re-runs the loader, it will ask permission to add the signature the
unsigned boot loader, boot.efi, to the authorised signatures database, meaning
Loader.efi will now no longer ask for present user authorisation every time
the system is started.


Creating, using and installing your own keys
============================================

To create PEM files with the certificate and the key for PK for example, do

openssl req -new -x509 -newkey rsa:2048 -subj "/CN=PK/" -keyout PK.key -out PK.crt -days 3650 -nodes -sha256

Which will create a self signed X509 certificate for PK in PK.crt (using
unprotected key PK.key with the subject common name PK (that's what the CN=PK
is doing).

You need to create at least three sets of certificates: one for PK, one for
KEK and one for db.

Now you need to take all the efi binaries in /usr/share/efitools/efi and sign
them with your own db key using

sbsign --key db.key --cert db.crt --output HelloWorld-signed.efi HelloWorld.efi

To install your new keys on the platform, first create your authorised update
bundles:

cert-to-sig-list PK.crt PK.esl
sign-efi-sig-list -k PK.key -c PK.crt PK PK.esl PK.auth

And repeat for KEK and db.  In setup mode, it only matters that the PK update
PK.auth is signed by the new platform key.  None of the other variables will
have their signatures checked.

Now on your platform update the variables, remembering to do PK last because
an update to PK usually puts the platform into secure mode

UpdateVars db db.auth
UpdateVars KEK KEK.auth
UpdateVars PK PK.auth

And you should now be running in secure mode with your own keys.
```
