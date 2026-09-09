# Brako Vault-Browsererweiterung — Datenschutzerklärung

[English](../../PRIVACY.md) | [Español](PRIVACY.es.md) | [Português](PRIVACY.pt.md) | [Français](PRIVACY.fr.md) | **[Deutsch](PRIVACY.de.md)** | [Italiano](PRIVACY.it.md) | [日本語](PRIVACY.ja.md) | [简体中文](PRIVACY.zh-CN.md)

Gültig ab: 9. September 2026

## Geltungsbereich

Diese Erklärung gilt für die Brako Vault-Erweiterung für Chromium-basierte
Browser und Firefox sowie für den optionalen Native-Messaging-Host unter
Windows.

## Verarbeitete Daten

Für ihren einzigen Zweck — Anmeldedaten aus einem lokal verschlüsselten Tresor
zu verwenden — verarbeitet die Erweiterung:

- Authentifizierungsdaten einschließlich Benutzernamen und Passwörtern;
- Ursprung und URL der aktiven Anmeldeseite;
- das ausdrückliche Absenden eines Formulars, um einen Speicher- oder
  Aktualisierungsvorschlag vorzubereiten;
- den Tresorordner und Gerätenamen aus den Einstellungen;
- vom Benutzer ausgewählte Tresoreinträge.

Die Erweiterung liest nur anmelderelevante Felder. Sie erfasst weder den
Verlauf anderer Tabs noch Cookies, Finanz- oder Gesundheitsdaten,
Werbe-IDs, Analysen, Absturzberichte oder Telemetrie.

## Verwendung

Die Daten werden nur verwendet, um den lokalen Tresor zu entsperren und zu
durchsuchen, passende Anmeldedaten anzuzeigen, ausgewählte Felder auszufüllen
und bestätigungspflichtige Speicher- oder Aktualisierungsvorschläge
vorzubereiten. Brako Vault sendet Website-Formulare niemals automatisch ab.

## Lokales Native Messaging

Die Erweiterung tauscht diese Daten mit `com.brakovault.desktop_host` aus,
einem separat installierten Windows-Programm. Firefox stuft Native Messaging
als Übertragung außerhalb des Browsers ein; deshalb deklariert das Manifest
Authentifizierungsinformationen, Browsing-Aktivität und Website-Aktivität.

Der Host arbeitet lokal und sendet keine Daten über das Netzwerk. Brako Vault,
Google, Mozilla und Dritte erhalten weder Tresor noch Anmeldedaten, URLs,
Gerätenamen, Ordnerpfade oder Nutzungsinformationen.

## Speicherung und Schutz

Der Tresor bleibt als verschlüsselte Datei `vault.bvda` im gewählten Ordner.
Der Host speichert seine Konfiguration unter
`%LOCALAPPDATA%\Brako Vault\config.json`. Bei aktiviertem Windows Hello-
Schnellzugriff liegt ein verschlüsselter Datensatz unter
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`; er enthält weder
Master-Passwort noch Master-Schlüssel im Klartext.

Der Tresor verwendet AES-256-GCM und Argon2id. Windows Hello überprüft vor dem
Schnellzugriff den lokalen Windows-Benutzer; es ersetzt die
Tresorverschlüsselung nicht und garantiert keinen hardwaregestützten Schutz.

## Weitergabe, Verkauf und Fernverarbeitung

Daten werden weder verkauft, vermietet oder an Dritte weitergegeben noch für
Werbung oder Kreditentscheidungen verwendet oder von einem Remote-Dienst
verarbeitet. Es gibt keine Benutzerkonten und keine Serversynchronisierung.

## Aufbewahrung und Löschung

Brako Vault kann Daten nicht aus der Ferne abrufen oder löschen:

- Das Deaktivieren von Windows Hello löscht `quick-unlock.json`.
- Die Deinstallation des Hosts entfernt Registrierungen und Schnellzugriff,
  bewahrt aber absichtlich `config.json` und den Tresor auf.
- Nach dem Schließen des Hosts kann der Benutzer `config.json` und den Tresor
  manuell löschen.
- Die Deinstallation der Erweiterung entfernt die vom Browser verwalteten
  Erweiterungsdaten.

Das Löschen des Tresors ist ohne weitere verschlüsselte Sicherung endgültig.

## Änderungen und Kontakt

Wesentliche Änderungen erscheinen unter derselben URL mit aktualisiertem
Datum. Für Datenschutz- oder Sicherheitsfragen nutzen Sie
[GitHub Private Vulnerability Reporting](https://github.com/waar19/brako-vault-releases/security/advisories/new).
