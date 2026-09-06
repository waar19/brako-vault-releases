# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | **[Italiano](README.it.md)** | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Repository ufficiale per il download di Brako Vault su Android.

Brako Vault è un gestore di password che funziona completamente
offline:

- L'APK non dichiara il permesso `INTERNET`.
- Non utilizza account, telemetria né sincronizzazione con server.
- Protegge la cassaforte con AES-256-GCM e una chiave derivata con
  Argon2id.
- La password principale non può essere recuperata. Se lo sblocco biometrico
  era già abilitato, resta valido e consente ancora di aprire la cassaforte:
  esporta immediatamente un `.bvda` cifrato con una nuova password per salvare
  i tuoi dati. Se la biometria non funziona, non era abilitata o viene
  invalidata prima dell'esportazione, la cassaforte diventa inaccessibile.
  Questo non recupera la password principale originale.

## Download e installazione

1. Apri la sezione [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Scarica soltanto `brako-vault-vX.Y.Z.apk`, `SHA256SUMS.txt` e
   `SIGNING-CERTIFICATE.txt` della versione più recente.
3. Installalo su un dispositivo Android con API 29 (Android 10) o
   superiore.
4. Conserva la tua password principale in un luogo sicuro. Non esiste
   alcun meccanismo di recupero.

> **Se stai arrivando da una build di sviluppo** (Android Studio che
> esegue `app` direttamente, o un APK firmato con la chiave debug di
> Android Studio), disinstallala **prima** di installare la build di
> release firmata. Le firme sono diverse e Android bloccherà
> l'aggiornamento con `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (gestori come
> Obtainium mostrano l'errore come "FailureConflict"). Passi:
>
> 1. Nella build di sviluppo, esporta una copia `.bvda` cifrata. Devi
>    inserire una password per il file esportato; può essere la password
>    principale o un'altra.
> 2. Salva il file fuori dallo spazio privato dell'app e verifica che
>    esista e non sia vuoto.
> 3. Solo allora vai in Impostazioni → App → Brako Vault →
>    Disinstalla. **La disinstallazione elimina i dati nello spazio
>    privato dell'app.**
> 4. Installa la build di release da questo repository e importa il
>    `.bvda` cifrato.
>
> **Se non hai un'esportazione confermata, non disinstallare la build di
> sviluppo.**

## APK vs AAB

- **APK** (Android Package): il file che installi su un dispositivo.
- **AAB** (Android App Bundle): il formato che si aspetta Google Play;
  contiene lo stesso codice, suddiviso in base alla configurazione del
  dispositivo. Non è possibile fare side-load di un `.aab`, per
  questo questo repository pubblica anche un `.apk`.

## Verificare la firma (dalla v0.4.0)

Dalla v0.4.0, ogni release pubblica
`SIGNING-CERTIFICATE.txt`. Verifica l'APK con gli strumenti ufficiali
Android:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'impronta SHA-256 mostrata da `apksigner` deve corrispondere a quella
in `SIGNING-CERTIFICATE.txt`. L'impronta SHA-256 ufficiale e canonica è:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

Dalla v0.4.0, sia il risultato di `apksigner` sia
`SIGNING-CERTIFICATE.txt` devono corrispondere esattamente a questa
impronta; il flusso di pubblicazione non va a buon fine se la firma è
diversa. Le versioni precedenti non includono questo file.

`SHA256SUMS.txt` e `SIGNING-CERTIFICATE.txt` sono pubblicati insieme ai
binari e non hanno una firma separata. I checksum rilevano corruzione o un
download incompleto, ma non una compromissione di GitHub. La firma dell'APK,
insieme all'impronta ancorata qui, autentica l'APK; questa garanzia non si
estende ai file AAB o BLF.

## Verificare il checksum SHA-256 (dalla v0.4.0)

Dalla v0.4.0, ogni release pubblica `SHA256SUMS.txt` con i digest di
ogni artefatto. Le versioni precedenti non includono questo file. Per
verificare su Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

Su macOS, verifica soltanto l'APK scaricato:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | shasum -a 256 -c -
```

Su Linux, verifica soltanto l'APK scaricato:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | sha256sum -c -
```

Il digest dell'APK deve corrispondere byte per byte alla relativa voce in
`SHA256SUMS.txt`. Se non corrisponde, non installare l'APK.

## Sugli archivi "Source code" generati automaticamente

GitHub produce automaticamente i link `Source code (zip)` e
`Source code (tar.gz)` in ogni release. Questi archivi sono generati a
partire da **questo** repository e contengono solo il README, l'avviso
di sicurezza e la configurazione del repository. **Non** contengono il
codice sorgente dell'applicazione, che è privato. Trattali come
documentazione, non come codice.

## Filtro password violate (opzionale)

Il database offline opzionale di password violate e i relativi checksum
sono pubblicati in
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Codice sorgente

Questo repository distribuisce esclusivamente binari ufficiali. Il
codice sorgente di Brako Vault non è pubblico e non è incluso in
questo repository.
