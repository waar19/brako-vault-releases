# Brako Vault — Sicherheit und Transparenz

[English](../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | **[Deutsch](SECURITY.de.md)** | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)**

Letzte Aktualisierung: 2026-09-05

Dieses Dokument beschreibt, wie Brako Vault die Daten auf Ihrem
Gerät schützt. Jede sachliche Aussage hier leitet sich aus der
öffentlichen
[kanonischen Kryptoparameterdatei](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md)
des Quellcode-Repositorys ab oder ist direkt an der veröffentlichten
APK und dem Anwendungsmanifest überprüfbar. Wo eine Aussage
„entworfen", aber nicht „auditiert" ist, sagt das Dokument das.

> **Der Quellcode ist privat.** Der Anwendungs-Quellcode liegt nicht
> in diesem Repository. Die Zahlen, Größen und Protokolle unten sind
> die maßgebliche öffentliche Aussage darüber, was die App tut. Die
> Release-APK wird aus demselben Code erstellt, der die Konstanten in
> `Argon2Spec` und `BvdaConstants` im privaten Repository enthält.

## 1. Verschlüsselung

| Was | Wie | Parameter |
|-----|-----|-----------|
| Symmetrische Verschlüsselung | AES-256-GCM (NIST SP 800-38D) | 256-Bit-Schlüssel, 128-Bit-Authentifizierungs-Tag, 96-Bit-Nonce (12 Byte) pro Schreibvorgang |
| Schlüsselableitung | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 Iterationen, 4 Lanes, 16-Byte-Salt, 32-Byte-abgeleiteter Schlüssel |
| Salt pro Tresor | Zufällig, 16 Byte | Bei Erstellung aus einer kryptographisch sicheren Zufallsquelle erzeugt |
| Nonce pro Schreibvorgang | Zufällig, 12 Byte | Einmalig pro Schreibvorgang, niemals mit demselben Schlüssel wiederverwendet |
| Header-Integrität | An den Geheimtext gebunden | 46 Byte des Headers als Additional Authenticated Data (AAD) authentifiziert |

**Was „Tag" bedeutet.** AES-GCM erzeugt ein 128-Bit-Authentifizierungs-Tag,
das an den Geheimtext angehängt wird. Jede Änderung der Datei (Magic,
KDF-Parameter, Salt, Nonce oder Geheimtext) lässt die Tag-Prüfung
fehlschlagen, und der Tresor verweigert das Öffnen.

**Was „Nonce" bedeutet.** Die 12-Byte-Nonce ist ein Wert, der unter
demselben Schlüssel niemals wiederholt werden darf. Die App erzeugt
bei jedem Speichern eine neue Zufalls-Nonce. Eine Nonce-Wiederverwendung
wäre katastrophal; deshalb schlägt die Implementierung explizit und
sicher fehl, wenn die Zufallsquelle defekt ist.

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

Die ersten 46 Byte des Headers sind das AAD. Das Tag ist Teil des
Geheimtextes, daher deckt der authentifizierte Inhalt der Datei alles
ab, mit Ausnahme des 4-Byte-Felds für die Länge des Geheimtextes.

## 3. Atomares Schreiben

Jede Speicherung folgt dem Muster Schreiben-dann-Umbenennen. Die App:

1. Schreibt den neuen Tresor nach `vault.bvda.tmp`.
2. Ruft `fsync` auf der temporären Datei auf.
3. Ruft `fsync` auf dem Verzeichnis auf.
4. Benennt `vault.bvda.tmp` atomar in `vault.bvda` um.

Wenn das Gerät zwischen den Schritten die Stromversorgung verliert
oder die App beendet wird, bleibt das vorherige `vault.bvda`
unangetastet, und die temporäre Datei bleibt als Müll zurück (wird
beim nächsten erfolgreichen Speichern bereinigt).

## 4. Biometrischer Schlüsselschutz

Wenn die biometrische Entsperrung aktiviert ist, wird der abgeleitete
Schlüssel mit einem im Android Keystore gespeicherten Schlüssel
umhüllt. Der durch Keystore geschützte Schlüssel:

- Verlässt die sichere Hardware nicht, wenn das Gerät über eine
  Trusted Execution Environment (TEE) oder einen StrongBox-Keymaster
  verfügt.
- Ist für Benutzer-Modus-Prozesse nicht extrahierbar.
- Wird ungültig, wenn der Benutzer alle Geräte-Biometrien entfernt,
  den Sperrbildschirm ändert oder das Gerät auf Werkseinstellungen
  zurücksetzt.

Die biometrische Abfrage wird vom Betriebssystem erzwungen; die App
kann sie nicht umgehen.

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
- Replay oder Manipulation der Tresordatei: Jede Bit-Änderung wird
  durch die AES-GCM-Tag-Prüfung erkannt.
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
- Dem Benutzer, der den Tresor freiwillig unverschlüsselt an einen
  Dritten exportiert.

## 7. Backups

Die App exportiert eine `.bvda`-Datei (dasselbe verschlüsselte Format
wie der Tresor auf der Festplatte). Die exportierte Datei wird mit
demselben aus dem Master-Passwort abgeleiteten Schlüssel verschlüsselt,
sofern der Benutzer nicht explizit „ohne Passwort exportieren"
wählt — und in diesem Fall warnt die App lautstark, dass die Datei
ungeschützt ist. Standardmäßig ist das geräteseitige Backup-System
deaktiviert, um den Tresor aus `adb backup`-Archiven und Cloud-Backups
fernzuhalten.

## 8. Offline-Synchronisation

Brako Vault hat keinen Server. Die Synchronisation zwischen Geräten
erfolgt durch den Austausch von `.bvda`-Dateien über einen vom
Benutzer gewählten Kanal (USB, E-Mail, Cloud-Speicher, AirDrop). Die
App öffnet niemals einen Netzwerk-Socket, daher kann der Kanal von
der App selbst nicht beobachtet werden. Die Datei wird durch dieselbe
AES-GCM-Tag-Prüfung Ende-zu-Ende authentifiziert, sodass ein
Dritter, der eine manipulierte Kopie weiterleitet, beim Import
erkannt wird.

## 9. Deklarierte Berechtigungen

Die APK deklariert genau drei Berechtigungen:

| Berechtigung | Warum |
|--------------|-------|
| `android.permission.CAMERA` | Zum Scannen von QR-Codes mit 2FA-Tokens. Die Kamera wird nur verwendet, wenn der Benutzer den QR-Scanner öffnet; Frames werden auf dem Gerät verarbeitet und niemals gespeichert. |
| `android.permission.VIBRATE` | Für haptisches Feedback bei Aktionen wie erfolgreichem Entsperren oder Kopieren in die Zwischenablage. |
| `android.permission.USE_BIOMETRIC` | Um optionale biometrische Entsperrung zu erlauben, vermittelt durch Android Keystore. Die App greift nicht direkt auf biometrische Daten zu. |

Die APK **deklariert nicht** `android.permission.INTERNET`. Es gibt
keinen Fallback, keine „nur Debug-Build"-Ausnahme und kein
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

Die Ausgabe darf nur die drei Berechtigungen aus §9 auflisten. Wenn
`android.permission.INTERNET` erscheint, ist die Datei nicht die
offizielle APK — installieren Sie sie nicht.

## 11. Herkunft und Signatur der Binärdateien

Jedes Release ist mit dem Release-Schlüssel des Maintainers signiert.
Der Fingerabdruck wird in den Release-Notes veröffentlicht. So
überprüfen Sie lokal:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

Der SHA-256-Fingerabdruck des Signaturzertifikats muss mit dem in
den Release-Notes übereinstimmen. Die Prüfsummen jedes Artefakts
befinden sich in `SHA256SUMS.txt` neben den Binärdateien desselben
Release.

## 12. Sicherheitsstufen

Brako Vault wird mit drei ausdrücklichen Sicherheitsstufen
ausgeliefert. Sie sind nicht dasselbe:

- **Entworfen.** Die Architektur und die Parameterentscheidungen in
  diesem Dokument sind das, was der Autor umzusetzen versprochen hat.
  Dies ist die schwächste Aussage.
- **Automatisch getestet.** Eine Test-Suite im privaten
  Quellcode-Repository — `CryptoSpecDocTest` und der Rest von
  `jvmTest` und `androidApp:testDebugUnitTest` — läuft bei jedem
  Push und überprüft, dass die Implementierung mit den hier
  gemachten Aussagen übereinstimmt, einschließlich der
  kryptographischen Parameter und der Abwesenheit von `INTERNET` im
  Manifest.
- **Extern auditiert.** Brako Vault wurde **nicht** von einer
  externen Stelle auditiert. Der Autor erhebt keinen Anspruch auf
  externe Zertifizierung, Common-Criteria-Bewertung oder
  Drittanbieter-Pentest. Wenn ein zukünftiges Audit durchgeführt
  wird, werden seine Ergebnisse hier mit Datum, Umfang und
  vollständigem Bericht veröffentlicht.

## 13. Meldung einer Schwachstelle

Wenn Sie eine Schwachstelle finden, schreiben Sie an
`security@brakovault.example` (ersetzen Sie diese durch die echte
Adresse, sobald das Projekt sie veröffentlicht). Eröffnen Sie
keine öffentliche GitHub-Issue für sicherheitsrelevante Meldungen.
Der Autor verpflichtet sich, innerhalb von 72 Stunden zu antworten
und innerhalb von 30 Tagen für bestätigte Probleme eine Korrektur
oder eine dokumentierte Risikoakzeptanz zu liefern.
