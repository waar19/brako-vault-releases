# Brako Vault Browser Extension — Privacy Policy

**[English](PRIVACY.md)** | [Español](docs/privacy/PRIVACY.es.md) | [Português](docs/privacy/PRIVACY.pt.md) | [Français](docs/privacy/PRIVACY.fr.md) | [Deutsch](docs/privacy/PRIVACY.de.md) | [Italiano](docs/privacy/PRIVACY.it.md) | [日本語](docs/privacy/PRIVACY.ja.md) | [简体中文](docs/privacy/PRIVACY.zh-CN.md)

Effective date: September 9, 2026

## Scope

This policy covers the Brako Vault browser extension for Chromium-based
browsers and Firefox, and its optional Windows native messaging host.

## Data handled

To provide its sole purpose—using credentials from a local encrypted
vault—the extension handles:

- authentication information, including usernames and passwords;
- the origin and URL of the active login page;
- an explicit form submission when preparing a save or update proposal;
- the vault folder and device name entered in Settings;
- vault entries selected by the user.

The extension only reads login-related fields. It does not collect browsing
history across tabs, cookies, payment data, health data, advertising
identifiers, analytics, crash reports, or telemetry.

## How data is used

Data is used only to unlock and search the local vault, show matching
credentials, fill the fields chosen by the user, and prepare a save or update
that requires confirmation. Brako Vault never submits a website form
automatically.

## Local native messaging

The extension exchanges the data above with
`com.brakovault.desktop_host`, a Windows program installed separately by the
user. Firefox classifies Native Messaging as transmission outside the browser;
this is why the Firefox manifest discloses authentication information,
browsing activity, and website activity.

The host is local. It does not send data over the network. Brako Vault, Google,
Mozilla, and third parties do not receive the vault, credentials, URLs, device
name, folder path, or usage information.

## Storage and protection

The vault remains in the user-selected folder as the encrypted `vault.bvda`
file. The host stores its configuration in
`%LOCALAPPDATA%\Brako Vault\config.json`. If Windows Hello quick unlock is
enabled, it stores an encrypted record in
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`; the record does not
contain the master password or master key in plaintext.

The vault uses AES-256-GCM and Argon2id. Windows Hello verifies the local
Windows user before quick unlock; it does not replace vault encryption or
guarantee hardware-backed protection.

## Sharing, sale, and remote processing

No data is sold, rented, shared with third parties, used for advertising,
used for credit decisions, or processed by a remote service. The product has
no user accounts and no server synchronization.

## Retention and deletion

Brako Vault cannot access or delete data remotely. Users control retention:

- disabling Windows Hello removes `quick-unlock.json`;
- uninstalling the host removes its registrations and quick-unlock record but
  intentionally preserves `config.json` and the user-selected vault;
- users can delete `config.json` and their vault manually after closing the
  host;
- uninstalling the extension removes the browser-managed extension data.

Deleting the vault is irreversible without another encrypted backup.

## Changes

Material changes to this policy will be published at this same URL and will
accompany an extension update. The effective date above will be updated.

## Contact

For privacy or security concerns, use
[GitHub Private Vulnerability Reporting](https://github.com/waar19/brako-vault-releases/security/advisories/new).
