# Brako Vault — Sicurezza e trasparenza

[English](../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | **[Italiano](SECURITY.it.md)** | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)**

Ultimo aggiornamento: 2026-09-05

Questo documento descrive come Brako Vault protegge i dati sul tuo
dispositivo. Ogni affermazione fattuale qui presente è derivata dal
[file canonico dei parametri crittografici](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md)
pubblico del repository del codice, oppure è verificabile direttamente
sull'APK pubblicato e sul manifest dell'applicazione. Dove
un'affermazione è "progettata" ma non "verificata da audit", il
documento lo dice.

> **Il codice sorgente è privato.** Il codice dell'applicazione non
> è in questo repository. I numeri, le dimensioni e i protocolli
> qui sotto sono la dichiarazione pubblica autorevole di ciò che fa
> l'app. L'APK di release è compilato a partire dallo stesso codice
> che contiene le costanti in `Argon2Spec` e `BvdaConstants` nel
> repository privato.

## 1. Cifratura

| Cosa | Come | Parametri |
|------|------|-----------|
| Cifratura simmetrica | AES-256-GCM (NIST SP 800-38D) | Chiave da 256 bit, tag di autenticazione da 128 bit, nonce da 96 bit (12 byte) per scrittura |
| Derivazione della chiave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iterazioni, 4 lane, salt da 16 byte, chiave derivata da 32 byte |
| Salt per cassaforte | Casuale, 16 byte | Generato da una fonte casuale crittograficamente sicura alla creazione |
| Nonce per scrittura | Casuale, 12 byte | Unico per scrittura, mai riutilizzato con la stessa chiave |
| Integrità dell'intestazione | Legata al ciphertext | 46 byte dell'intestazione autenticati come Additional Authenticated Data (AAD) |

**Cosa significa "tag".** AES-GCM produce un tag di autenticazione
da 128 bit che viene accodato al ciphertext. Qualsiasi modifica del
file (magic, parametri KDF, salt, nonce o ciphertext) fa fallire la
verifica del tag, e la cassaforte rifiuta di aprirsi.

**Cosa significa "nonce".** Il nonce da 12 byte è un valore che non
deve mai ripetersi sotto la stessa chiave. L'app genera un nonce
casuale nuovo a ogni salvataggio. Riutilizzare un nonce sarebbe
catastrofico; per questo l'implementazione fallisce in modo sicuro
ed esplicito se la fonte casuale è guasta.

## 2. Formato del file

Tutti i dati della cassaforte sono memorizzati in un unico file:
`vault.bvda`.

Layout (big-endian per i campi multi-byte, eccetto `saltLen` e
`nonceLen` che sono 1 byte):

```
 offset  dim  campo
 ------  ---  ------
   0      4   magic            = "BVDA" (ASCII)
   4      1   formatVersion    = 0x01
   5      1   kdfId            = 0x01 (Argon2id)
   6      1   aeadId           = 0x01 (AES-256-GCM)
   7      1   kdfParallelism   = 0x04
   8      4   kdfMemoryKiB     BE = 65 536
  12      4   kdfIterations    BE = 3
  16      1   saltLen          = 0x10 (16)
  17     16   kdfSalt
  33      1   nonceLen         = 0x0C (12)
  34     12   aeadNonce
  46      4   ciphertextLen    BE  <- NON fa parte dell'AAD
  50      N   ciphertext (N = ciphertextLen, include tag da 16 B)
```

I primi 46 byte dell'intestazione sono l'AAD. Il tag fa parte del
ciphertext, quindi il contenuto autenticato del file copre tutto
tranne il campo da 4 byte con la lunghezza del ciphertext.

## 3. Scrittura atomica

Ogni salvataggio segue lo schema scrivi-poi-rinomina. L'app:

1. Scrive la nuova cassaforte in `vault.bvda.tmp`.
2. Chiama `fsync` sul file temporaneo.
3. Chiama `fsync` sulla directory.
4. Rinomina atomicamente `vault.bvda.tmp` in `vault.bvda`.

Se il dispositivo perde alimentazione o l'app viene terminata tra i
passi, il precedente `vault.bvda` rimane intatto e il file
temporaneo resta come spazzatura (pulita al prossimo salvataggio
riuscito).

## 4. Protezione biometrica della chiave

Quando lo sblocco biometrico è attivo, la chiave derivata è
incapsulata con una chiave conservata nell'Android Keystore. La
chiave protetta dal Keystore:

- Non lascia mai l'hardware sicuro quando il dispositivo dispone di
  un Trusted Execution Environment (TEE) o di uno StrongBox Keymaster.
- Non è estraibile dai processi in modalità utente.
- Viene invalidata quando l'utente rimuove tutte le impronte
  biometriche del dispositivo, cambia la schermata di blocco, o
  effettua un reset di fabbrica.

La richiesta biometrica è imposta dal sistema operativo; l'app non
può bypassarla.

## 5. Nessun recupero della password

Non esiste alcun meccanismo di recupero, né reset via email, né
chiave di recupero, né servizio clienti che possa aprire una
cassaforte. La password principale è l'unica chiave, non viene mai
memorizzata, mai trasmessa, mai registrata. Se perdi la password
principale, la cassaforte è persa.

L'autore di questo progetto non può aprire la tua cassaforte.

## 6. Modello di minaccia — cosa protegge Brako Vault

L'app è progettata per proteggere da:

- Furto del dispositivo spento, da parte di un avversario senza la
  password principale e senza la capacità di aggirare la
  crittografia del disco di Android.
- Attacchi via rete: l'app non ha il permesso `INTERNET`, quindi
  condizioni di rete compromesse non possono esfiltrare la
  cassaforte.
- Replay o modifica del file della cassaforte: qualsiasi cambio di
  un bit è rilevato dalla verifica del tag AES-GCM.
- Forza bruta sulla password principale: Argon2id con 64 MiB e 3
  iterazioni rende ogni tentativo costoso; un attaccante offline
  deve comunque indovinare la password.

L'app **non** è progettata per proteggere da:

- Un dispositivo compromesso o rooted in esecuzione mentre la
  cassaforte è aperta. Una volta verificata la password principale,
  la chiave è in memoria e il processo ha accesso al testo in
  chiaro.
- Sguardi indiscreti, keylogger o malware in esecuzione con lo
  stesso UID dell'app.
- Coercizione: un attaccante determinato con accesso fisico a un
  dispositivo sbloccato può leggere la cassaforte.
- Una password principale debole o riutilizzata. Argon2id rallenta
  l'attacco ma non rende sicuro "123456".
- L'utente che esporta volontariamente la cassaforte non cifrata a
  un terzo.

## 7. Backup

L'app esporta un file `.bvda` (lo stesso formato cifrato della
cassaforte su disco). Il file esportato è cifrato con la stessa
chiave derivata dalla password principale, a meno che l'utente non
scelga esplicitamente "esporta senza password" — e in quel caso
l'app avvisa con decisione che il file non sarà protetto. Per
impostazione predefinita, il sistema di backup del dispositivo è
disattivato per tenere la cassaforte fuori dagli archivi `adb
backup` e dai backup cloud.

## 8. Sincronizzazione offline

Brako Vault non ha un server. La sincronizzazione tra dispositivi
avviene scambiando file `.bvda` attraverso un canale scelto
dall'utente (USB, email, archiviazione cloud, AirDrop). L'app non
apre mai un socket di rete, quindi il canale non può essere
osservato dall'app stessa. Il file è autenticato end-to-end dalla
stessa verifica del tag AES-GCM, quindi un terzo che inoltra una
copia alterata viene rilevato all'importazione.

## 9. Permessi dichiarati

L'APK dichiara esattamente tre permessi:

| Permesso | Perché |
|----------|--------|
| `android.permission.CAMERA` | Per scansionare codici QR contenenti token 2FA. La fotocamera è usata solo quando l'utente apre lo scanner QR; i frame sono elaborati sul dispositivo e mai memorizzati. |
| `android.permission.VIBRATE` | Per il feedback aptico su azioni come sblocco riuscito o copia negli appunti. |
| `android.permission.USE_BIOMETRIC` | Per consentire lo sblocco biometrico opzionale, mediato da Android Keystore. L'app non accede direttamente ai dati biometrici. |

L'APK **non** dichiara `android.permission.INTERNET`. Non c'è
fallback, non c'è eccezione "solo build debug" e nessun SDK di
terze parti lo richiede. Se una funzionalità futura avesse bisogno
di accesso alla rete, la funzionalità viene riconsiderata; il
permesso non viene aggiunto.

## 10. Come verificare che l'APK non ha il permesso INTERNET

Non devi fidarti di questo documento. Puoi verificare su qualsiasi
dispositivo o workstation:

```shell
# Dai build-tools di Android SDK:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# oppure
aapt dump permissions brako-vault-vX.Y.Z.apk
```

L'output deve elencare solo i tre permessi del §9. Se appare
`android.permission.INTERNET`, il file non è l'APK ufficiale — non
installarlo.

## 11. Provenienza e firma dei binari

Ogni release è firmata con la chiave di release del mantenitore.
L'impronta è pubblicata nelle note di release. Per verificare
localmente:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'impronta SHA-256 del certificato di firma deve corrispondere a
quella delle note di release. I checksum di ogni artefatto sono in
`SHA256SUMS.txt`, accanto ai binari della stessa release.

## 12. Livelli di garanzia

Brako Vault viene offerto con tre livelli di garanzia espliciti. Non
sono la stessa cosa:

- **Progettato.** L'architettura e le scelte dei parametri di
  questo documento sono ciò che l'autore si è impegnato a
  implementare. È l'affermazione più debole.
- **Testato automaticamente.** Una suite di test nel repository
  privato del codice — `CryptoSpecDocTest` e il resto di `jvmTest`
  e `androidApp:testDebugUnitTest` — viene eseguita a ogni push e
  verifica che l'implementazione corrisponda a quanto dichiarato
  qui, inclusi i parametri crittografici e l'assenza di `INTERNET`
  nel manifest.
- **Verificato da audit esterno.** Brako Vault **non** è stato
  verificato da una parte esterna. L'autore non rivendica alcuna
  certificazione esterna, valutazione Common Criteria, né
  penetration test di terze parti. Se in futuro verrà eseguito un
  audit, i suoi risultati saranno pubblicati qui con data, ambito e
  rapporto completo.

## 13. Segnalazione di una vulnerabilità

Se trovi una vulnerabilità, scrivi a
`security@brakovault.example` (sostituisci con l'indirizzo reale
quando il progetto lo renderà pubblico). Non aprire una issue
pubblica su GitHub per segnalazioni sensibili. L'autore si impegna
ad accusare ricevuta entro 72 ore e a fornire una correzione o
un'accettazione documentata del rischio entro 30 giorni per i
problemi confermati.
