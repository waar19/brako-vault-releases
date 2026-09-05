# Brako Vault — Sécurité et transparence

[English](../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | **[Français](SECURITY.fr.md)** | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)**

Dernière mise à jour : 2026-09-05

Ce document décrit comment Brako Vault protège les données sur votre
appareil. Chaque affirmation factuelle ici est dérivée du
[fichier canonique des paramètres cryptographiques](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md)
public du dépôt de code, ou vérifiable directement sur l'APK publié et
le manifeste de l'application. Lorsqu'une affirmation est « conçue »
mais pas « auditée », le document le dit.

> **Le code source est privé.** Le code de l'application n'est pas
> dans ce dépôt. Les nombres, tailles et protocoles ci-dessous
> constituent la déclaration publique faisant foi de ce que fait
> l'application. L'APK de release est compilé à partir du même code
> qui contient les constantes dans `Argon2Spec` et `BvdaConstants`
> du dépôt privé.

## 1. Chiffrement

| Quoi | Comment | Paramètres |
|------|---------|------------|
| Chiffrement symétrique | AES-256-GCM (NIST SP 800-38D) | Clé de 256 bits, tag d'authentification de 128 bits, nonce de 96 bits (12 octets) par écriture |
| Dérivation de clé | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 itérations, 4 lanes, sel de 16 octets, clé dérivée de 32 octets |
| Sel par coffre | Aléatoire, 16 octets | Généré par une source aléatoire cryptographiquement sûre à la création |
| Nonce par écriture | Aléatoire, 12 octets | Unique par écriture, jamais réutilisé avec la même clé |
| Intégrité de l'en-tête | Liée au ciphertext | 46 octets d'en-tête authentifiés comme Additional Authenticated Data (AAD) |

**Ce que veut dire « tag ».** AES-GCM produit un tag d'authentification
de 128 bits qui est concaténé au ciphertext. Toute modification du
fichier (magic, paramètres KDF, sel, nonce ou ciphertext) fait échouer
la vérification du tag, et le coffre refuse de s'ouvrir.

**Ce que veut dire « nonce ».** Le nonce de 12 octets est une valeur
qui ne doit jamais se répéter sous la même clé. L'application génère
un nouveau nonce aléatoire à chaque sauvegarde. Réutiliser un nonce
serait catastrophique ; c'est pourquoi l'implémentation échoue
explicitement et de manière sûre si la source aléatoire est défaillante.

## 2. Format de fichier

Toutes les données du coffre sont stockées dans un seul fichier :
`vault.bvda`.

Disposition (big-endian pour les champs multi-octets, sauf `saltLen` et
`nonceLen` qui font 1 octet) :

```
 offset  tail.  champ
 ------  -----  -----
   0       4    magic            = "BVDA" (ASCII)
   4       1    formatVersion    = 0x01
   5       1    kdfId            = 0x01 (Argon2id)
   6       1    aeadId           = 0x01 (AES-256-GCM)
   7       1    kdfParallelism   = 0x04
   8       4    kdfMemoryKiB     BE = 65 536
  12       4    kdfIterations    BE = 3
  16       1    saltLen          = 0x10 (16)
  17      16    kdfSalt
  33       1    nonceLen         = 0x0C (12)
  34      12    aeadNonce
  46       4    ciphertextLen    BE  <- NE fait PAS partie de l'AAD
  50       N    ciphertext (N = ciphertextLen, inclut le tag de 16 o)
```

Les 46 premiers octets de l'en-tête constituent l'AAD. Le tag fait
partie du ciphertext, donc le contenu authentifié du fichier couvre
tout, sauf le champ de 4 octets portant la longueur du ciphertext.

## 3. Écriture atomique

Chaque sauvegarde suit le motif écrire-puis-renommer. L'application :

1. Écrit le nouveau coffre dans `vault.bvda.tmp`.
2. Appelle `fsync` sur le fichier temporaire.
3. Appelle `fsync` sur le répertoire.
4. Renomme atomiquement `vault.bvda.tmp` en `vault.bvda`.

Si l'appareil perd l'alimentation ou si l'application est tuée entre
les étapes, le `vault.bvda` précédent reste intact et le fichier
temporaire reste comme déchet (nettoyé à la prochaine sauvegarde
réussie).

## 4. Protection de la clé biométrique

Lorsque le déverrouillage biométrique est activé, la clé dérivée est
emballée avec une clé stockée dans l'Android Keystore. La clé gardée
par Keystore :

- Ne quitte jamais le matériel sécurisé lorsque l'appareil dispose
  d'un Trusted Execution Environment (TEE) ou d'un StrongBox
  Keymaster.
- N'est pas extractible par les processus en mode utilisateur.
- Est invalidée lorsque l'utilisateur retire toutes les données
  biométriques, change l'écran de verrouillage, ou effectue une
  réinitialisation d'usine.

L'invite biométrique est imposée par le système d'exploitation ;
l'application ne peut pas la contourner.

## 5. Pas de récupération du mot de passe

Il n'existe aucun mécanisme de récupération, ni réinitialisation par
e-mail, ni clé de secours, ni service client capable d'ouvrir un
coffre. Le mot de passe maître est la seule clé, il n'est jamais
stocké, jamais transmis, jamais journalisé. Si vous perdez le mot de
passe maître, le coffre est perdu.

L'auteur de ce projet ne peut pas non plus ouvrir votre coffre.

## 6. Modèle de menace — ce que Brako Vault protège

L'application est conçue pour protéger contre :

- Le vol de l'appareil éteint, par un adversaire qui n'a ni le mot de
  passe maître ni la capacité de contourner le chiffrement de disque
  d'Android.
- Les attaques réseau : l'application n'a pas la permission
  `INTERNET`, donc des conditions réseau compromises ne peuvent pas
  exfiltrer le coffre.
- Le rejeu ou la modification du fichier du coffre : tout
  changement d'un bit est détecté par la vérification du tag
  AES-GCM.
- La force brute sur le mot de passe maître : Argon2id avec 64 MiB
  et 3 itérations rend chaque tentative coûteuse ; un attaquant hors
  ligne doit encore deviner le mot de passe.

L'application **n'est pas** conçue pour protéger contre :

- Un appareil compromis ou rooté qui tourne pendant que le coffre est
  ouvert. Une fois le mot de passe maître vérifié, la clé est en
  mémoire et le processus a accès au texte clair.
- Le regard indiscret, les enregistreurs de frappe ou un malware
  s'exécutant avec le même UID que l'application.
- La coercition : un attaquant déterminé ayant un accès physique à
  un appareil déverrouillé peut lire le coffre.
- Un mot de passe maître faible ou réutilisé. Argon2id ralentit
  l'attaque mais ne rend pas « 123456 » sûr.
- L'utilisateur qui exporterait volontairement le coffre non chiffré
  à un tiers.

## 7. Sauvegardes

L'application exporte un fichier `.bvda` (le même format chiffré que
le coffre sur disque). Le fichier exporté est chiffré avec la même
clé dérivée du mot de passe maître, sauf si l'utilisateur choisit
explicitement « exporter sans mot de passe » — et dans ce cas,
l'application avertit clairement que le fichier sera non protégé. Par
défaut, le système de sauvegarde de l'appareil est désactivé pour
garder le coffre hors des archives `adb backup` et des sauvegardes
cloud.

## 8. Synchronisation hors ligne

Brako Vault n'a pas de serveur. La synchronisation entre appareils
fonctionne en échangeant des fichiers `.bvda` par un canal choisi par
l'utilisateur (USB, e-mail, stockage cloud, AirDrop). L'application
n'ouvre jamais de socket réseau, donc le canal ne peut pas être
observé par l'application elle-même. Le fichier est authentifié de
bout en bout par la même vérification du tag AES-GCM ; ainsi, un tiers
qui relaie une copie altérée est détecté à l'importation.

## 9. Permissions déclarées

L'APK déclare exactement trois permissions :

| Permission | Pourquoi |
|------------|----------|
| `android.permission.CAMERA` | Pour scanner des QR codes contenant des jetons 2FA. La caméra n'est utilisée que lorsque l'utilisateur ouvre le scanner QR ; les images sont traitées sur l'appareil et jamais stockées. |
| `android.permission.VIBRATE` | Pour le retour haptique lors d'actions comme un déverrouillage réussi ou la copie dans le presse-papiers. |
| `android.permission.USE_BIOMETRIC` | Pour autoriser le déverrouillage biométrique optionnel, médié par Android Keystore. L'application n'accède pas directement aux données biométriques. |

L'APK **ne déclare pas** `android.permission.INTERNET`. Il n'y a pas
de solution de repli, pas d'exception « build debug uniquement » et
aucun SDK tiers ne la demande. Si une fonctionnalité future avait
besoin d'un accès réseau, la fonctionnalité serait reconsidérée ; la
permission n'est pas ajoutée.

## 10. Comment vérifier que l'APK n'a pas la permission INTERNET

Vous n'avez pas besoin de faire confiance à ce document. Vous pouvez
vérifier sur n'importe quel appareil ou poste de travail :

```shell
# Depuis Android SDK build-tools :
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# ou
aapt dump permissions brako-vault-vX.Y.Z.apk
```

La sortie ne doit lister que les trois permissions de la §9. Si
`android.permission.INTERNET` apparaît, le fichier n'est pas l'APK
officiel — ne l'installez pas.

## 11. Provenance et signature des binaires

Chaque release est signé avec la clé de release du mainteneur.
L'empreinte est publiée dans les notes de release. Pour vérifier
localement :

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'empreinte SHA-256 du certificat de signature doit correspondre à
celle des notes de release. Les sommes de contrôle de chaque artefact
se trouvent dans `SHA256SUMS.txt`, à côté des binaires du même
release.

## 12. Niveaux d'assurance

Brako Vault est proposé avec trois niveaux d'assurance explicites. Ce
n'est pas la même chose :

- **Conçu.** L'architecture et les choix de paramètres de ce document
  sont ce que l'auteur s'est engagé à mettre en œuvre. C'est
  l'affirmation la plus faible.
- **Testé automatiquement.** Une suite de tests dans le dépôt de
  code privé — `CryptoSpecDocTest` et le reste de `jvmTest` et
  `androidApp:testDebugUnitTest` — s'exécute à chaque push et vérifie
  que l'implémentation correspond à ce qui est déclaré ici, y
  compris les paramètres cryptographiques et l'absence d'`INTERNET`
  dans le manifeste.
- **Audité en externe.** Brako Vault **n'a pas** été audité par un
  tiers. L'auteur ne revendique aucune certification externe, aucune
  évaluation common criteria ni aucun test d'intrusion tiers. Si un
  audit futur est réalisé, ses conclusions seront publiées ici avec
  la date, le périmètre et le rapport complet.

## 13. Signalement d'une vulnérabilité

Si vous découvrez une vulnérabilité, écrivez à
`security@brakovault.example` (remplacez par la véritable adresse
lorsque le projet la rendra publique). N'ouvrez pas d'issue publique
sur GitHub pour les signalements sensibles. L'auteur s'engage à
accuser réception sous 72 heures et à fournir un correctif ou une
acceptation documentée du risque sous 30 jours pour les problèmes
confirmés.
