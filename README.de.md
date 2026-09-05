# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | **[Deutsch](README.de.md)** | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Offizielles Download-Repository für Brako Vault auf Android.

Brako Vault ist ein Passwort-Manager, der vollständig offline
funktioniert:

- Die APK deklariert die Berechtigung `INTERNET` nicht.
- Es werden keine Konten, Telemetrie oder Server-Synchronisation
  verwendet.
- Der Tresor wird mit AES-256-GCM und einem mit Argon2id abgeleiteten
  Schlüssel geschützt.
- Das Master-Passwort kann nicht wiederhergestellt werden. Wenn Sie es
  verlieren, ist der Tresor verloren.

## Herunterladen und installieren

1. Öffnen Sie den Bereich [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Laden Sie die `.apk`-Datei der aktuellen Version herunter.
3. Installieren Sie sie auf einem Android-Gerät mit API 29 (Android 10)
   oder höher.
4. Bewahren Sie Ihr Master-Passwort an einem sicheren Ort auf. Es gibt
   keinen Wiederherstellungsmechanismus.

> **Wenn Sie von einer Entwicklungsversion kommen** (Android Studio, das
> `app` direkt ausführt, oder eine APK, die mit dem Debug-Schlüssel
> von Android Studio signiert ist), deinstallieren Sie diese **bevor**
> Sie die signierte Release-Version installieren. Die Signaturen
> unterscheiden sich und Android blockiert das Upgrade mit
> `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (Manager wie Obtainium zeigen den
> Fehler als „FailureConflict"). Schritte:
>
> 1. Exportieren Sie in der Entwicklungsversion eine verschlüsselte
>    `.bvda`-Sicherung. Für die exportierte Datei müssen Sie ein Passwort
>    eingeben; es kann das Master-Passwort oder ein anderes sein.
> 2. Speichern Sie die Datei außerhalb des privaten App-Speichers und
>    bestätigen Sie, dass sie vorhanden und nicht leer ist.
> 3. Gehen Sie erst dann zu Einstellungen → Apps → Brako Vault →
>    Deinstallieren. **Bei der Deinstallation werden die Daten im
>    privaten App-Speicher gelöscht.**
> 4. Installieren Sie die Release-Version aus diesem Repository und
>    importieren Sie die verschlüsselte `.bvda`.
>
> **Wenn Sie keinen bestätigten Export haben, deinstallieren Sie die
> Entwicklungsversion nicht.**

## APK vs AAB

- **APK** (Android Package): die Datei, die Sie auf einem Gerät
  installieren.
- **AAB** (Android App Bundle): das Format, das Google Play erwartet;
  es enthält denselben Code, aufgeteilt nach Gerätekonfiguration. Ein
  `.aab` lässt sich nicht direkt installieren, weshalb dieses
  Repository zusätzlich eine `.apk` veröffentlicht.

## Signatur prüfen (ab v0.4.0)

Ab v0.4.0 veröffentlicht jedes Release
`SIGNING-CERTIFICATE.txt`. Prüfen Sie die APK mit den offiziellen
Android-Werkzeugen:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

Der von `apksigner` ausgegebene SHA-256-Fingerabdruck muss mit dem in
`SIGNING-CERTIFICATE.txt` übereinstimmen. Frühere Versionen enthalten
diese Datei nicht.

## SHA-256-Prüfsumme verifizieren (ab v0.4.0)

Ab v0.4.0 veröffentlicht jedes Release `SHA256SUMS.txt` mit den
Prüfsummen jedes Artefakts. Frühere Versionen enthalten diese Datei
nicht. So prüfen Sie unter Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

Unter macOS / Linux:

```shell
sha256sum -c SHA256SUMS.txt
```

Die veröffentlichten Prüfsummen müssen Byte für Byte mit denen in
`SHA256SUMS.txt` übereinstimmen. Falls nicht, installieren Sie die
Datei nicht.

## Über die automatisch erzeugten „Source code"-Archive

GitHub erzeugt auf jedem Release automatisch die Links `Source code
(zip)` und `Source code (tar.gz)`. Diese Archive werden aus
**diesem** Repository erzeugt und enthalten nur die README, den
Sicherheitshinweis und die Repository-Konfiguration. Sie enthalten
**nicht** den Anwendungs-Quellcode, der privat ist. Betrachten Sie sie
als Dokumentation, nicht als Code.

## Filter für geleakte Passwörter (optional)

Die optionale Offline-Datenbank geleakter Passwörter und ihre
Prüfsummen werden in
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases)
veröffentlicht.

## Quellcode

Dieses Repository vertreibt ausschließlich offizielle Binärdateien. Der
Brako-Vault-Quellcode ist nicht öffentlich und nicht in diesem
Repository enthalten.
