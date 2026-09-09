# Estensione browser Brako Vault — Informativa sulla privacy

[English](../../PRIVACY.md) | [Español](PRIVACY.es.md) | [Português](PRIVACY.pt.md) | [Français](PRIVACY.fr.md) | [Deutsch](PRIVACY.de.md) | **[Italiano](PRIVACY.it.md)** | [日本語](PRIVACY.ja.md) | [简体中文](PRIVACY.zh-CN.md)

Data di entrata in vigore: 9 settembre 2026

## Ambito

Questa informativa riguarda l’estensione Brako Vault per browser basati su
Chromium e Firefox e il relativo host di messaggistica nativa opzionale per
Windows.

## Dati trattati

Per il suo unico scopo — usare credenziali da una cassaforte locale cifrata —
l’estensione tratta:

- dati di autenticazione, inclusi nomi utente e password;
- origine e URL della pagina di accesso attiva;
- invio esplicito di un modulo per preparare una proposta di salvataggio o
  aggiornamento;
- cartella della cassaforte e nome del dispositivo inseriti nelle Impostazioni;
- voci della cassaforte selezionate dall’utente.

L’estensione legge solo campi relativi all’accesso. Non raccoglie cronologia
tra schede, cookie, dati finanziari o sanitari, identificatori pubblicitari,
analisi, segnalazioni di arresto anomalo o telemetria.

## Uso dei dati

I dati sono usati solo per sbloccare e cercare nella cassaforte locale,
mostrare credenziali corrispondenti, compilare i campi scelti e preparare un
salvataggio o aggiornamento che richiede conferma. Brako Vault non invia mai
automaticamente un modulo del sito.

## Messaggistica nativa locale

L’estensione scambia questi dati con `com.brakovault.desktop_host`, un
programma Windows installato separatamente dall’utente. Firefox considera
Native Messaging una trasmissione esterna al browser; per questo il manifest
dichiara informazioni di autenticazione, attività di navigazione e attività
sui siti web.

L’host è locale e non invia dati in rete. Brako Vault, Google, Mozilla e terze
parti non ricevono la cassaforte, credenziali, URL, nome del dispositivo,
percorso della cartella o informazioni d’uso.

## Archiviazione e protezione

La cassaforte resta nella cartella scelta come file cifrato `vault.bvda`.
L’host salva la configurazione in
`%LOCALAPPDATA%\Brako Vault\config.json`. Se è attivo lo sblocco rapido con
Windows Hello, salva un record cifrato in
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`; non contiene password
principale o chiave principale in chiaro.

La cassaforte usa AES-256-GCM e Argon2id. Windows Hello verifica l’utente
Windows locale prima dello sblocco rapido; non sostituisce la cifratura della
cassaforte e non garantisce protezione hardware.

## Condivisione, vendita e trattamento remoto

Nessun dato viene venduto, affittato, condiviso con terzi, usato per pubblicità
o decisioni creditizie, né elaborato da servizi remoti. Il prodotto non ha
account utente né sincronizzazione server.

## Conservazione ed eliminazione

Brako Vault non può accedere o cancellare dati da remoto:

- disattivare Windows Hello elimina `quick-unlock.json`;
- disinstallare l’host elimina registrazioni e accesso rapido, ma conserva
  intenzionalmente `config.json` e la cassaforte;
- l’utente può eliminare manualmente `config.json` e la cassaforte dopo aver
  chiuso l’host;
- disinstallare l’estensione elimina i dati gestiti dal browser.

Eliminare la cassaforte è irreversibile senza un altro backup cifrato.

## Modifiche e contatti

Le modifiche sostanziali saranno pubblicate allo stesso URL con la data
aggiornata. Per questioni di privacy o sicurezza, usa la
[segnalazione privata delle vulnerabilità di GitHub](https://github.com/waar19/brako-vault-releases/security/advisories/new).
