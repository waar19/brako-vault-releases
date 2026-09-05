# Brako Vault

**[English](README.md)** | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Official download repository for Brako Vault on Android.

Brako Vault is a password manager that works completely offline:

- The APK does not declare the `INTERNET` permission.
- It uses no accounts, telemetry, or server synchronization.
- It protects the vault with AES-256-GCM and a key derived with Argon2id.
- The master password cannot be recovered. If you lose it, the vault is gone.

## Download and install

1. Open the [Releases](https://github.com/waar19/brako-vault-releases/releases) section.
2. Download the `.apk` file of the latest version.
3. Install it on an Android device with API 29 (Android 10) or higher.
4. Keep your master password in a safe place. There is no recovery mechanism.

> **If you are coming from a development build** (Android Studio running
> `app` directly, or an APK signed with the Android Studio debug key),
> uninstall it **before** installing the release-signed build. The
> signatures are different and Android will block the upgrade with
> `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (managers such as Obtainium show
> the error as "FailureConflict"). Steps:
>
> 1. In the development build, export an encrypted `.bvda` backup. You
>    must enter a password for the exported file; it may be your master
>    password or a different one.
> 2. Save the exported file outside the app's private storage and confirm
>    that it exists and is not empty.
> 3. Only then go to Settings → Apps → Brako Vault → Uninstall.
>    **Uninstalling deletes the data in the app's private storage.**
> 4. Install the release build from this repository and import the
>    encrypted `.bvda`.
>
> **If you do not have a confirmed export, do not uninstall the
> development build.**

The `.aab` file is published for distribution and validation, but it is
not installed directly on a device.

## APK vs AAB

- **APK** (Android Package): the file you install on a device.
- **AAB** (Android App Bundle): the format Google Play expects; contains
  the same code split per device configuration. Side-loading an `.aab`
  is not supported, which is why this repository publishes an `.apk`
  alongside it.

## Verify the signature (v0.4.0 and later)

Starting with v0.4.0, each release publishes
`SIGNING-CERTIFICATE.txt`. Check the APK with the official Android
tools:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

The SHA-256 fingerprint printed by `apksigner` must match the one in
`SIGNING-CERTIFICATE.txt`. Earlier releases do not include this file.

## Verify the SHA-256 checksum (v0.4.0 and later)

Starting with v0.4.0, each release publishes `SHA256SUMS.txt` with the
digests of every artifact. Earlier releases do not include this file.
To verify on Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

On macOS / Linux:

```shell
sha256sum -c SHA256SUMS.txt
```

The published digests must match the ones in `SHA256SUMS.txt` byte for
byte. If they do not, do not install the file.

## About the auto-generated "Source code" archives

GitHub automatically produces `Source code (zip)` and `Source code (tar.gz)`
links on every release. Those archives are built from **this** repository
and contain only the README, the security notice, and the repository
configuration. They do **not** contain the application source code, which
is private. Treat them as documentation, not as code.

## Leaked-password filter (optional)

The optional offline database of leaked passwords and its checksums are
published in
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Source code

This repository distributes only official binaries. The Brako Vault source
code is not public and is not included in this repository.
