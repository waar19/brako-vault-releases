# Brako Vault — Sicurezza e trasparenza

[English](../../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | **[Italiano](SECURITY.it.md)** | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)

Ultimo aggiornamento: 2026-09-05

Questo documento autonomo è la dichiarazione pubblica di sicurezza e
trasparenza di Brako Vault. Il repository del codice è privato; questo
repository pubblico di release non fornisce il sorgente né afferma che
terzi possano riprodurre la build. Firma e permessi dell'APK possono
essere verificati manualmente. I limiti delle garanzie sono nella §12.

## 1. Cifratura

| Cosa | Come | Parametri |
|------|------|-----------|
| Cifratura simmetrica | AES-256-GCM (NIST SP 800-38D) | Chiave da 32 byte (256 bit), tag da 16 byte (128 bit), nonce da 12 byte (96 bit) per scrittura |
| Derivazione della chiave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iterazioni, parallelismo 4, salt da 16 byte, chiave da 32 byte |
| Salt per cassaforte | Casuale, 16 byte | Generato da una fonte casuale crittograficamente sicura alla creazione |
| Nonce per scrittura | Casuale, 12 byte | A ogni scrittura vengono richiesti 12 nuovi byte a `SecureRandom` di Java |
| Intestazione | 50 byte | I primi 46 byte sono AAD; i 4 byte finali di `ciphertextLen` non sono autenticati |

**Cosa significa "tag".** AES-GCM produce un tag da 16 byte che
autentica i primi 46 byte come AAD e il ciphertext dichiarato.
`ciphertextLen` non è autenticato. Il lettore attuale permette e ignora
i byte successivi al ciphertext dichiarato; il tag non li copre.

**Cosa significa "nonce".** Il nonce da 12 byte è un valore che non
deve ripetersi sotto la stessa chiave. A ogni scrittura l'app richiede
12 nuovi byte a `SecureRandom` di Java. Non esiste un rilevatore
esplicito di ripetizioni né un controllo della fonte: è una protezione
probabilistica, non una garanzia assoluta o fail-closed.

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

La regione autenticata comprende i primi 46 byte, il ciphertext
dichiarato e il tag GCM. `ciphertextLen` non è autenticato; i byte
successivi sono permessi e ignorati.

## 3. Sostituzione in scrittura e limiti di durabilità

Ogni salvataggio segue lo schema scrivi-poi-rinomina. L'app:

1. Scrive la nuova cassaforte in `vault.bvda.tmp`.
2. Sincronizza il descrittore del file temporaneo.
3. Tenta `ATOMIC_MOVE` con sostituzione.
4. Poi tenta uno spostamento non atomico con sostituzione.
5. Infine copia sulla destinazione ed elimina il temporaneo.

La directory non è sincronizzata. Solo il primo metodo mira
all'atomicità, secondo il filesystem. I fallback non la garantiscono.
Un guasto può lasciare una destinazione incompleta o assente; non si
promette durabilità totale.

## 4. Protezione biometrica della chiave

La chiave derivata è incapsulata con una chiave Android Keystore non
esportabile tramite API, che richiede `BIOMETRIC_STRONG` e viene
invalidata al nuovo enrolment biometrico.

Il supporto hardware, TEE o StrongBox dipende dal dispositivo. L'app
non richiede StrongBox né verifica l'hardware backing; tali proprietà
non vanno presunte su tutti i dispositivi.

## 5. Nessun recupero della password

Non esiste alcun meccanismo di recupero, né reset via email, né
chiave di recupero, né servizio clienti che possa aprire una
cassaforte. La password principale originale non viene mai memorizzata,
trasmessa o registrata e non può essere recuperata.

Se lo sblocco biometrico era già abilitato e resta valido, può ancora
aprire la cassaforte. Esporta immediatamente un `.bvda` cifrato con una
nuova password per salvare i tuoi dati; questo non recupera la password
principale originale. Se la biometria non era abilitata, non funziona o
viene invalidata prima dell'esportazione, l'accesso alla cassaforte è
perso definitivamente.

L'autore di questo progetto non può aprire la tua cassaforte.

## 6. Modello di minaccia — cosa protegge Brako Vault

L'app è progettata per proteggere da:

- Furto del dispositivo spento, da parte di un avversario senza la
  password principale e senza la capacità di aggirare la
  crittografia del disco di Android.
- Attacchi via rete: l'app non ha il permesso `INTERNET`, quindi
  condizioni di rete compromesse non possono esfiltrare la
  cassaforte.
- Modifica del contenuto BVDA autenticato: sono rilevati cambiamenti ai
  primi 46 byte, al ciphertext dichiarato o al tag GCM.
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
- Replay o rollback a un `.bvda` valido precedente: AES-GCM non prova
  freschezza o monotonicità di versione.
- Modifiche a `ciphertextLen` o ai byte successivi non autenticati.
- Divulgazione della password di export o uso di canali non affidabili.

## 7. Backup

Ogni export `.bvda` richiede una password inserita dall'utente, che può
essere quella principale o un'altra. Non esiste export senza password.
Il backup del dispositivo è disattivato per impostazione predefinita.

## 8. Sincronizzazione offline

Brako Vault non ha un server. La sincronizzazione tra dispositivi
avviene scambiando file `.bvda` attraverso un canale scelto
dall'utente (USB, email, archiviazione cloud, AirDrop). L'app non
apre mai un socket di rete, quindi il canale non può essere
osservato dall'app stessa. All'importazione AES-GCM verifica la regione
BVDA autenticata. Non rileva replay di vecchi file validi né dati non
autenticati descritti nella §2.

## 9. Permessi dichiarati

Dalla v0.4.0, l'APK dichiara esattamente tre permessi della piattaforma
Android:

| Permesso | Perché |
|----------|--------|
| `android.permission.CAMERA` | Per scansionare codici QR contenenti token 2FA. La fotocamera è usata solo quando l'utente apre lo scanner QR; i frame sono elaborati sul dispositivo e mai memorizzati. |
| `android.permission.VIBRATE` | Per il feedback aptico su azioni come sblocco riuscito o copia negli appunti. |
| `android.permission.USE_BIOMETRIC` | Per consentire lo sblocco biometrico opzionale, mediato da Android Keystore. L'app non accede direttamente ai dati biometrici. |

Il workflow di release verifica con `aapt2` questo insieme esatto di
permessi della piattaforma. AndroidX aggiunge inoltre il permesso
personalizzato
`com.brakovault.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`. È un
permesso privato, con protezione a livello di firma; non è un permesso
della piattaforma Android e non concede accesso alla rete. Nella v0.3.0
potevano comparire ulteriori permessi normali aggiunti da AndroidX.

L'APK **non dichiarava e continua a non dichiarare**
`android.permission.INTERNET`. Non c'è fallback, non c'è eccezione
"solo build debug" e nessun SDK di terze parti lo richiede. Se una
funzionalità futura avesse bisogno di accesso alla rete, la funzionalità
verrebbe riconsiderata; il permesso non verrebbe aggiunto.

## 10. Come verificare che l'APK non ha il permesso INTERNET

Non devi fidarti di questo documento. Puoi verificare su qualsiasi
dispositivo o workstation:

```shell
# Dai build-tools di Android SDK:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# oppure
aapt dump permissions brako-vault-vX.Y.Z.apk
```

Nella v0.4.0 e successive, i permessi attesi della piattaforma Android
sono esattamente i tre del §9. `aapt2` può mostrare anche il permesso
privato di AndroidX descritto in quella sezione. Nella v0.3.0 potevano
comparire altri permessi normali aggiunti da AndroidX. Se
`android.permission.INTERNET` compare in qualsiasi versione, l'APK non
corrisponde a questo documento; non installarlo.

## 11. Provenienza e firma dei binari

Ogni release è firmata con la chiave del mantenitore. Dalla v0.4.0
include anche `SIGNING-CERTIFICATE.txt` e `SHA256SUMS.txt`; le versioni
precedenti no. Ispezione locale:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'impronta SHA-256 ufficiale e canonica del certificato di firma è:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

Dalla v0.4.0, il workflow di release richiede questa impronta.
L'impronta SHA-256 mostrata da `apksigner` deve corrispondere sia al
valore precedente sia a `SIGNING-CERTIFICATE.txt`, mentre gli hash degli
artefatti devono corrispondere a `SHA256SUMS.txt`.

`SHA256SUMS.txt` e `SIGNING-CERTIFICATE.txt` non hanno una firma
separata. Consentono di rilevare la corruzione dei file, ma da soli non
possono rilevare la compromissione di un account o repository GitHub,
perché un attaccante potrebbe sostituirli insieme agli artefatti. La
firma dell'APK e l'impronta ancorata sopra autenticano l'APK; non
autenticano un file AAB o BLF.

## 12. Livelli di garanzia

Le evidenze hanno ambiti distinti:

- **Dichiarazione di progetto.** Questo documento pubblico indica
  progetto e parametri esatti.
- **Test interni.** I test privati verificano parametri in un documento
  privato e comportamento interno, non questi documenti pubblici né
  l'APK/manifest pubblicato.
- **Verifica esterna manuale.** Certificato e permessi sono ispezionabili
  con i comandi sopra; dalla v0.4.0 anche hash e impronta.
  Possono essere confrontati con i due file della release e con
  l'impronta canonica di questo documento, entro i limiti indicati nel
  §11.

Brako Vault **non** ha ricevuto audit esterni, certificazioni, Common
Criteria o penetration test di terzi. La build non è dichiarata
riproducibile.

## 13. Segnalazione di una vulnerabilità

Per segnalazioni sensibili usa la
[segnalazione privata](https://github.com/waar19/brako-vault-releases/security/advisories/new).
Non inserire dettagli sensibili in issue pubbliche. Per domande non
sensibili usa le [issue pubbliche](https://github.com/waar19/brako-vault-releases/issues).
Non sono promesse scadenze fisse di risposta o correzione.
