# Brako Vault — Sicherheit und Transparenz

[English](../../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | **[Deutsch](SECURITY.de.md)** | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)

Letzte Aktualisierung: 2026-09-05

Dieses eigenständige Dokument ist die öffentliche Sicherheits- und
Transparenzerklärung für Brako Vault. Das Quellcode-Repository ist
privat; dieses öffentliche Release-Repository stellt weder Quellcode
bereit noch behauptet es reproduzierbare Builds durch Dritte. Signatur
und Berechtigungen der APK können manuell geprüft werden. Die Grenzen
der Nachweise stehen in §12.

## 1. Verschlüsselung

| Was | Wie | Parameter |
|-----|-----|-----------|
| Symmetrische Verschlüsselung | AES-256-GCM (NIST SP 800-38D) | 32-Byte-Schlüssel (256 Bit), 16-Byte-Tag (128 Bit), 12-Byte-Nonce (96 Bit) pro Schreibvorgang |
| Schlüsselableitung | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 Iterationen, Parallelität 4, 16-Byte-Salt, 32-Byte-Schlüssel |
| Salt pro Tresor | Zufällig, 16 Byte | Bei Erstellung aus einer kryptographisch sicheren Zufallsquelle erzeugt |
| Nonce pro Schreibvorgang | Zufällig, 12 Byte | Pro Schreibvorgang werden 12 neue Byte von Java `SecureRandom` angefordert |
| Header | 50 Byte | Die ersten 46 Byte sind AAD; die letzten 4 Byte `ciphertextLen` sind nicht authentifiziert |

**Was „Tag" bedeutet.** AES-GCM erzeugt ein 16-Byte-Tag. Es
authentifiziert die ersten 46 Header-Byte als AAD und den deklarierten
Geheimtext. `ciphertextLen` ist nicht authentifiziert. Der aktuelle
Reader erlaubt und ignoriert Byte hinter dem deklarierten Geheimtext;
das Tag deckt sie nicht ab.

**Was „Nonce" bedeutet.** Die 12-Byte-Nonce ist ein Wert, der unter
demselben Schlüssel nicht wiederholt werden darf. Pro Schreibvorgang
fordert die App 12 neue Byte von Java `SecureRandom` an. Es gibt weder
einen expliziten Wiederholungsdetektor noch eine Prüfung auf eine
defekte Quelle: Dies ist probabilistischer Schutz, keine absolute
Eindeutigkeits- oder Fail-closed-Garantie.

## 2. Dateiformat

Alle Tresordaten werden in einer einzigen Datei gespeichert:
`vault.bvda`.

Aufbau (Big-Endian für Multi-Byte-Felder, außer `saltLen` und
`nonceLen`, die 1 Byte sind):

```
 Offset  Gr.   Feld
 ------  ---   ----
   0      4    magic            = "BVDA" (ASCII)
   4      1    formatVersion    = 0x01
   5      1    kdfId            = 0x01 (Argon2id)
   6      1    aeadId           = 0x01 (AES-256-GCM)
   7      1    kdfParallelism   = 0x04
   8      4    kdfMemoryKiB     BE = 65 536
  12      4    kdfIterations    BE = 3
  16      1    saltLen          = 0x10 (16)
  17     16    kdfSalt
  33      1    nonceLen         = 0x0C (12)
  34     12    aeadNonce
  46      4    ciphertextLen    BE  <- KEIN Teil des AAD
  50      N    ciphertext (N = ciphertextLen, enthält 16-Byte-Tag)
```

Authentifiziert sind die ersten 46 Byte sowie der deklarierte Geheimtext
mit GCM-Tag. `ciphertextLen` ist nicht authentifiziert; nachfolgende
Byte werden erlaubt und ignoriert.

## 3. Ersetzung und Haltbarkeitsgrenzen

Jede Speicherung folgt dem Muster Schreiben-dann-Umbenennen. Die App:

1. Schreibt den neuen Tresor nach `vault.bvda.tmp`.
2. Synchronisiert den Deskriptor der temporären Datei.
3. Versucht `ATOMIC_MOVE` mit Ersetzung.
4. Versucht danach eine nicht atomare Verschiebung mit Ersetzung.
5. Kopiert als letzten Fallback über das Ziel und löscht die Temp-Datei.

Das Verzeichnis wird nicht synchronisiert. Nur der erste Weg soll
atomar sein, abhängig vom Dateisystem. Die Fallbacks garantieren keine
Atomarität. Ein Ausfall kann ein unterbrochenes oder fehlendes Ziel
hinterlassen; vollständige Crash-Haltbarkeit wird nicht versprochen.

## 4. Biometrischer Schlüsselschutz

Der abgeleitete Schlüssel wird mit einem Android-Keystore-Schlüssel
umhüllt. Dieser ist über die Android-API nicht exportierbar, erfordert
`BIOMETRIC_STRONG` und wird bei erneuter Biometrie-Registrierung
ungültig.

Hardware-, TEE- oder StrongBox-Schutz hängt vom Gerät ab. Die App
fordert StrongBox nicht an und prüft Hardware-Backing nicht; diese
Eigenschaften gelten daher nicht garantiert auf jedem Gerät.

## 5. Keine Passwort-Wiederherstellung

Es gibt keinen Wiederherstellungsmechanismus, keine E-Mail-Zurücksetzung,
keinen Wiederherstellungsschlüssel, keinen Kundensupport, der einen
Tresor öffnen kann. Das Master-Passwort ist der einzige Schlüssel; es
wird niemals gespeichert, niemals übertragen und niemals protokolliert.
Wenn Sie das Master-Passwort verlieren, ist der Tresor verloren.

Der Autor dieses Projekts kann Ihren Tresor ebenfalls nicht öffnen.

## 6. Bedrohungsmodell — was Brako Vault schützt

Die App ist so konzipiert, dass sie schützt vor:

- Diebstahl des ausgeschalteten Geräts durch einen Angreifer ohne
  Master-Passwort und ohne Möglichkeit, die Datenträgerverschlüsselung
  von Android zu umgehen.
- Netzwerkbasierten Angriffen: Die App hat keine `INTERNET`-Berechtigung,
  sodass kompromittierte Netzbedingungen den Tresor nicht exfiltrieren
  können.
- Manipulation authentifizierter BVDA-Inhalte: Änderungen der ersten 46
  Byte, des deklarierten Geheimtexts oder GCM-Tags werden erkannt.
- Brute-Force des Master-Passworts: Argon2id mit 64 MiB und 3
  Iterationen macht jeden Versuch teuer; ein Offline-Angreifer muss
  immer noch das Passwort erraten.

Die App ist **nicht** dafür konzipiert, zu schützen vor:

- Einem kompromittierten oder gerooteten Gerät, das läuft, während
  der Tresor geöffnet ist. Sobald das Master-Passwort verifiziert ist,
  befindet sich der Schlüssel im Speicher, und der Prozess hat Zugriff
  auf Klartext.
- Shoulder-Surfing, Keyloggern oder Malware, die mit derselben UID
  wie die App läuft.
- Nötigung: Ein entschlossener Angreifer mit physischem Zugriff auf
  ein entsperrtes Gerät kann den Tresor lesen.
- Einem schwachen oder wiederverwendeten Master-Passwort. Argon2id
  verlangsamt das Raten, kann aber „123456" nicht sicher machen.
- Replay oder Rollback auf eine ältere gültige `.bvda`; AES-GCM beweist
  weder Aktualität noch monotone Versionen.
- Änderungen an `ciphertextLen` oder nicht authentifizierten Folgebytes.
- Offenlegung des Exportpassworts oder Übertragung über unsichere Kanäle.

## 7. Backups

Jeder `.bvda`-Export erfordert ein vom Benutzer eingegebenes Passwort.
Es kann das Master-Passwort oder ein anderes sein; einen passwortlosen
Export gibt es nicht. Geräte-Backups sind standardmäßig deaktiviert.

## 8. Offline-Synchronisation

Brako Vault hat keinen Server. Die Synchronisation zwischen Geräten
erfolgt durch den Austausch von `.bvda`-Dateien über einen vom
Benutzer gewählten Kanal (USB, E-Mail, Cloud-Speicher, AirDrop). Die
App öffnet niemals einen Netzwerk-Socket, daher kann der Kanal von
der App selbst nicht beobachtet werden. Beim Import prüft AES-GCM den
authentifizierten BVDA-Bereich. Das erkennt weder Replay älterer
gültiger Dateien noch nicht authentifizierte Daten aus §2.

## 9. Deklarierte Berechtigungen

Ab v0.4.0 deklariert die APK genau drei
Android-Plattformberechtigungen:

| Berechtigung | Warum |
|--------------|-------|
| `android.permission.CAMERA` | Zum Scannen von QR-Codes mit 2FA-Tokens. Die Kamera wird nur verwendet, wenn der Benutzer den QR-Scanner öffnet; Frames werden auf dem Gerät verarbeitet und niemals gespeichert. |
| `android.permission.VIBRATE` | Für haptisches Feedback bei Aktionen wie erfolgreichem Entsperren oder Kopieren in die Zwischenablage. |
| `android.permission.USE_BIOMETRIC` | Um optionale biometrische Entsperrung zu erlauben, vermittelt durch Android Keystore. Die App greift nicht direkt auf biometrische Daten zu. |

Der Release-Workflow prüft diesen exakten Satz von
Plattformberechtigungen mit `aapt2`. AndroidX fügt außerdem die
benutzerdefinierte Berechtigung
`com.brakovault.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION` hinzu.
Sie ist privat und signaturgeschützt, keine Android-Plattformberechtigung
und gewährt keinen Netzwerkzugriff. Bei v0.3.0 konnten zusätzliche
normale, von AndroidX eingebrachte Berechtigungen angezeigt werden.

Die APK **hatte und hat keine** `android.permission.INTERNET`-Berechtigung.
Es gibt keinen Fallback, keine „nur Debug-Build"-Ausnahme und kein
Drittanbieter-SDK, das sie anfordert. Wenn eine zukünftige Funktion
Netzwerkzugriff benötigen würde, wird die Funktion überdacht; die
Berechtigung wird nicht hinzugefügt.

## 10. So überprüfen Sie, dass die APK keine INTERNET-Berechtigung hat

Sie müssen diesem Dokument nicht vertrauen. Sie können auf jedem
Gerät oder Arbeitsplatz überprüfen:

```shell
# Von den Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# oder
aapt dump permissions brako-vault-vX.Y.Z.apk
```

Ab v0.4.0 sind genau die drei in §9 genannten
Android-Plattformberechtigungen zu erwarten. `aapt2` kann zusätzlich die
dort beschriebene private AndroidX-Berechtigung anzeigen. Bei v0.3.0
konnten weitere normale, von AndroidX eingebrachte Berechtigungen
erscheinen. Wenn `android.permission.INTERNET` in irgendeiner Version
erscheint, entspricht die APK nicht diesem Dokument; installieren Sie
sie nicht.

## 11. Herkunft und Signatur der Binärdateien

Jedes Release ist mit dem Maintainer-Schlüssel signiert. Ab v0.4.0
enthält es auch `SIGNING-CERTIFICATE.txt` und `SHA256SUMS.txt`; frühere
Versionen nicht. Lokale Prüfung:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

Der offizielle, kanonische SHA-256-Fingerabdruck des Signaturzertifikats
lautet:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

Ab v0.4.0 verlangt der Release-Workflow diesen Fingerabdruck. Der von
`apksigner` angezeigte SHA-256-Fingerabdruck muss sowohl mit dem obigen
Wert als auch mit `SIGNING-CERTIFICATE.txt` übereinstimmen; die
Artefakt-Hashes müssen `SHA256SUMS.txt` entsprechen.

`SHA256SUMS.txt` und `SIGNING-CERTIFICATE.txt` haben keine separate
Signatur. Sie können Beschädigungen erkennen, aber für sich allein keine
Kompromittierung eines GitHub-Kontos oder -Repositorys, da ein Angreifer
sie zusammen mit den Artefakten ersetzen könnte. Die APK-Signatur und
der oben verankerte Fingerabdruck authentifizieren die APK; eine AAB-
oder BLF-Datei authentifizieren sie nicht.

## 12. Sicherheitsstufen

Die Nachweise haben unterschiedliche Reichweite:

- **Designaussage.** Dieses öffentliche Dokument nennt Design und Werte.
- **Interne Tests.** Private Tests prüfen kanonische Parameter in einem
  privaten Dokument und internes Verhalten, nicht diese öffentlichen
  Dokumente oder die veröffentlichte APK/das Manifest.
- **Externe manuelle Prüfung.** Zertifikat und Berechtigungen sind mit
  obigen Befehlen prüfbar; ab v0.4.0 lassen sich auch Hashes und
  Fingerabdruck mit den beiden Release-Dateien und dem kanonischen
  Fingerabdruck dieses Dokuments vergleichen, unter den in §11
  genannten Einschränkungen.

Brako Vault hat **kein** externes Sicherheitsaudit, keine Zertifizierung,
Common-Criteria-Bewertung oder Drittanbieter-Pentest erhalten. Eine
Reproduzierbarkeit des Builds wird nicht garantiert.

## 13. Meldung einer Schwachstelle

Nutzen Sie für sensible Meldungen die
[private Schwachstellenmeldung](https://github.com/waar19/brako-vault-releases/security/advisories/new).
Keine sensiblen Details in öffentlichen Issues. Nicht sensible Fragen
können in [öffentlichen Issues](https://github.com/waar19/brako-vault-releases/issues)
gestellt werden. Feste Reaktions- oder Behebungsfristen werden nicht
versprochen.
