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
- Le mot de passe maître ne peut pas être récupéré. Si le déverrouillage
  biométrique était déjà activé, il reste valide et permet encore d'ouvrir le
  coffre : exportez immédiatement un `.bvda` chiffré avec un nouveau mot de
  passe pour sauver vos données. Si la biométrie ne fonctionne pas, n'était
  pas activée ou devient invalide avant l'exportation, le coffre devient
  inaccessible. Cela ne récupère pas le mot de passe maître d'origine.

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
> 1. Dans la build de développement, exportez une copie `.bvda`
>    chiffrée. Vous devez saisir un mot de passe pour le fichier exporté ;
>    il peut s'agir du mot de passe maître ou d'un autre.
> 2. Enregistrez le fichier hors du stockage privé de l'application et
>    confirmez qu'il existe et n'est pas vide.
> 3. Alors seulement, allez dans Paramètres → Applications →
>    Brako Vault → Désinstaller. **La désinstallation efface les données
>    du stockage privé de l'application.**
> 4. Installez la build de release de ce dépôt et importez le `.bvda`
>    chiffré.
>
> **Si vous n'avez pas d'exportation confirmée, ne désinstallez pas la
> build de développement.**

## APK vs AAB

- **APK** (Android Package) : le fichier que vous installez sur un
  appareil.
- **AAB** (Android App Bundle) : le format attendu par Google Play ;
  il contient le même code, découpé selon la configuration de
  l'appareil. Le side-load d'un `.aab` n'est pas supporté ; c'est
  pourquoi ce dépôt publie un `.apk` en plus de l'`.aab`.

## Vérifier la signature (v0.4.0 et versions ultérieures)

À partir de la v0.4.0, chaque release publie
`SIGNING-CERTIFICATE.txt`. Vérifiez l'APK avec les outils officiels
d'Android :

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'empreinte SHA-256 affichée par `apksigner` doit correspondre à celle
de `SIGNING-CERTIFICATE.txt`. L'empreinte SHA-256 officielle et canonique
est :

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

À partir de la v0.4.0, le résultat d'`apksigner` et
`SIGNING-CERTIFICATE.txt` doivent tous deux correspondre exactement à cette
empreinte ; le processus de publication échoue si la signature diffère. Les
versions antérieures ne contiennent pas ce fichier.

`SHA256SUMS.txt` et `SIGNING-CERTIFICATE.txt` sont publiés avec les binaires
et ne disposent pas d'une signature distincte. Les sommes de contrôle
détectent une corruption ou un téléchargement incomplet, mais pas une
compromission de GitHub. La signature de l'APK associée à l'empreinte ancrée
ici authentifie l'APK ; cette garantie ne s'étend pas aux fichiers AAB ou
BLF.

## Vérifier la somme de contrôle SHA-256 (v0.4.0 et versions ultérieures)

À partir de la v0.4.0, chaque release publie `SHA256SUMS.txt` avec les
empreintes de chaque artefact. Les versions antérieures ne contiennent
pas ce fichier. Pour vérifier sous Windows (PowerShell) :

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

Sous macOS :

```shell
shasum -a 256 -c SHA256SUMS.txt
```

Sous Linux :

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
