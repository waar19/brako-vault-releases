# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | **[Français](README.fr.md)** | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Dépôt officiel de téléchargement de Brako Vault pour Android.

Brako Vault est un gestionnaire de mots de passe qui fonctionne
intégralement hors ligne :

- L'APK ne déclare pas la permission `INTERNET`.
- Il n'utilise aucun compte, aucune télémétrie, aucune synchronisation
  avec un serveur.
- Il protège le coffre avec AES-256-GCM et une clé dérivée via
  Argon2id.
- Le mot de passe maître ne peut pas être récupéré. Si vous le perdez,
  le coffre est perdu.

## Télécharger et installer

1. Ouvrez la section [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Téléchargez le fichier `.apk` de la dernière version.
3. Installez-le sur un appareil Android avec l'API 29 (Android 10) ou
   supérieure.
4. Conservez votre mot de passe maître en lieu sûr. Il n'existe aucun
   mécanisme de récupération.

> **Si vous venez d'une build de développement** (Android Studio
> exécutant `app` directement, ou un APK signé avec la clé debug
> d'Android Studio), désinstallez-la **avant** d'installer la build
> signée de release. Les signatures sont différentes et Android bloquera
> la mise à jour avec `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (les
> gestionnaires comme Obtainium affichent l'erreur comme
> "FailureConflict"). Étapes :
>
> 1. Paramètres → Applications → Brako Vault → Désinstaller.
> 2. Installer la build de release depuis ce dépôt.
> 3. Si vous aviez un coffre, réimportez le `.bvda` (il n'est pas
>    copié entre signatures).

## APK vs AAB

- **APK** (Android Package) : le fichier que vous installez sur un
  appareil.
- **AAB** (Android App Bundle) : le format attendu par Google Play ;
  il contient le même code, découpé selon la configuration de
  l'appareil. Le side-load d'un `.aab` n'est pas supporté ; c'est
  pourquoi ce dépôt publie un `.apk` en plus de l'`.aab`.

## Vérifier la signature

Vous pouvez vérifier la signature de l'APK avec les outils officiels
d'Android :

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

## Vérifier la somme de contrôle SHA-256

Chaque release publie un fichier `SHA256SUMS.txt` avec les empreintes
de chaque artefact. Pour vérifier sous Windows (PowerShell) :

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

Sous macOS / Linux :

```shell
sha256sum -c SHA256SUMS.txt
```

Les empreintes publiées doivent correspondre octet par octet à celles
du `SHA256SUMS.txt`. Si ce n'est pas le cas, n'installez pas le
fichier.

## À propos des archives « Source code » générées automatiquement

GitHub produit automatiquement les liens `Source code (zip)` et
`Source code (tar.gz)` sur chaque release. Ces archives sont construites
à partir de **ce** dépôt et ne contiennent que le README, l'avis de
sécurité et la configuration du dépôt. Elles **ne** contiennent **pas**
le code source de l'application, qui est privé. Traitez-les comme de la
documentation, pas comme du code.

## Filtre de mots de passe fuités (optionnel)

La base hors ligne optionnelle de mots de passe fuités et ses sommes
de contrôle sont publiées dans
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Code source

Ce dépôt distribue uniquement les binaires officiels. Le code source de
Brako Vault n'est pas public et n'est pas inclus dans ce dépôt.
