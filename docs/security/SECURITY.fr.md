# Brako Vault — Sécurité et transparence

[English](../../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | **[Français](SECURITY.fr.md)** | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)

Dernière mise à jour : 2026-09-05

Ce document autonome est la déclaration publique de sécurité et de
transparence de Brako Vault. Le dépôt du code de l'application est
privé ; ce dépôt public de releases ne fournit pas le code source et
n'affirme pas qu'un tiers peut reproduire la build. L'APK publié et son
manifeste permettent de vérifier manuellement certaines affirmations,
notamment l'identité de signature et les permissions. Les limites
d'assurance figurent en §12.

## 1. Chiffrement

| Quoi | Comment | Paramètres |
|------|---------|------------|
| Chiffrement symétrique | AES-256-GCM (NIST SP 800-38D) | Clé de 32 octets (256 bits), tag de 16 octets (128 bits), nonce de 12 octets (96 bits) par écriture |
| Dérivation de clé | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 itérations, parallélisme 4, sel de 16 octets, clé dérivée de 32 octets |
| Sel par coffre | Aléatoire, 16 octets | Généré par une source aléatoire cryptographiquement sûre à la création |
| Nonce par écriture | Aléatoire, 12 octets | 12 nouveaux octets demandés à `SecureRandom` de Java à chaque écriture |
| En-tête | 50 octets | Les 46 premiers sont authentifiés comme AAD ; les 4 octets finaux de `ciphertextLen` ne le sont pas |

**Ce que veut dire « tag ».** AES-GCM produit un tag de 16 octets. Il
authentifie les 46 premiers octets de l'en-tête comme AAD et le
ciphertext déclaré. Une modification de ce contenu échoue. Les 4 octets
de `ciphertextLen` ne sont pas authentifiés ; le lecteur actuel autorise
et ignore les octets après le ciphertext déclaré, hors couverture du tag.

**Ce que veut dire « nonce ».** Le nonce de 12 octets est une valeur
qui ne doit jamais se répéter sous la même clé. À chaque écriture,
l'application demande 12 nouveaux octets à `SecureRandom` de Java. Elle
n'a ni détecteur explicite de répétition ni test de source défaillante :
c'est une protection probabiliste, pas une garantie d'unicité absolue
ni un contrôle fail-closed.

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

La région authentifiée comprend les 46 premiers octets, le ciphertext
déclaré et son tag GCM. `ciphertextLen` n'est pas authentifié. Le lecteur
actuel autorise et ignore les octets supplémentaires.

## 3. Remplacement à l'écriture et limites de durabilité

Chaque sauvegarde suit le motif écrire-puis-renommer. L'application :

1. Écrit le nouveau coffre dans `vault.bvda.tmp`.
2. Synchronise le descripteur du fichier temporaire.
3. Tente `ATOMIC_MOVE` avec remplacement.
4. À défaut, tente un déplacement non atomique avec remplacement.
5. À défaut, copie sur la destination puis supprime le temporaire.

Le répertoire n'est pas synchronisé. Seule la première méthode vise
l'atomicité, selon le système de fichiers. Les solutions de repli ne la
garantissent pas. Une panne peut laisser une destination interrompue ou
absente ; aucune durabilité totale n'est promise.

## 4. Protection de la clé biométrique

La clé dérivée est emballée avec une clé Android Keystore non exportable
via l'API Android, exigeant `BIOMETRIC_STRONG` et invalidée lors d'un
nouvel enrôlement biométrique.

Le support matériel, TEE ou StrongBox dépend de l'appareil. L'application
ne demande pas StrongBox et ne vérifie pas le support matériel ; ces
propriétés ne doivent pas être supposées sur tous les appareils.

## 5. Pas de récupération du mot de passe

Il n'existe aucun mécanisme de récupération, ni réinitialisation par
e-mail, ni clé de secours, ni service client capable d'ouvrir un
coffre. Le mot de passe maître d'origine n'est jamais stocké, transmis
ou journalisé et ne peut pas être récupéré.

Si le déverrouillage biométrique était déjà activé et reste valide, il
peut encore ouvrir le coffre. Exportez immédiatement un `.bvda` chiffré
avec un nouveau mot de passe pour sauver vos données ; cela ne récupère
pas le mot de passe maître d'origine. Si la biométrie n'était pas
activée, échoue ou devient invalide avant l'exportation, l'accès au
coffre est définitivement perdu.

L'auteur de ce projet ne peut pas non plus ouvrir votre coffre.

## 6. Modèle de menace — ce que Brako Vault protège

L'application est conçue pour protéger contre :

- Le vol de l'appareil éteint, par un adversaire qui n'a ni le mot de
  passe maître ni la capacité de contourner le chiffrement de disque
  d'Android.
- Les attaques réseau : l'application n'a pas la permission
  `INTERNET`, donc des conditions réseau compromises ne peuvent pas
  exfiltrer le coffre.
- La modification du contenu BVDA authentifié : les changements des 46
  premiers octets, du ciphertext déclaré ou du tag GCM sont détectés.
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
- Le rejeu ou retour à un ancien `.bvda` valide. AES-GCM n'établit ni
  fraîcheur ni progression de version.
- Les changements de `ciphertextLen` ou des octets supplémentaires non
  authentifiés.
- La divulgation du mot de passe d'un export ou un canal non fiable.

## 7. Sauvegardes

Chaque export `.bvda` exige un mot de passe saisi par l'utilisateur,
qui peut être le mot de passe maître ou un autre. Il n'existe aucun
export sans mot de passe. La sauvegarde de l'appareil est désactivée par
défaut pour exclure le coffre d'`adb backup` et du cloud.

## 8. Synchronisation hors ligne

Brako Vault n'a pas de serveur. La synchronisation entre appareils
fonctionne en échangeant des fichiers `.bvda` par un canal choisi par
l'utilisateur (USB, e-mail, stockage cloud, AirDrop). L'application
n'ouvre jamais de socket réseau, donc le canal ne peut pas être
observé par l'application elle-même. À l'importation, AES-GCM vérifie le
contenu BVDA authentifié et détecte les modifications de cette région.
Cela ne détecte pas le rejeu d'un ancien fichier valide ni les données
non authentifiées décrites en §2.

## 9. Permissions déclarées

À partir de la v0.4.0, l'APK déclare exactement trois permissions de la
plateforme Android :

| Permission | Pourquoi |
|------------|----------|
| `android.permission.CAMERA` | Pour scanner des QR codes contenant des jetons 2FA. La caméra n'est utilisée que lorsque l'utilisateur ouvre le scanner QR ; les images sont traitées sur l'appareil et jamais stockées. |
| `android.permission.VIBRATE` | Pour le retour haptique lors d'actions comme un déverrouillage réussi ou la copie dans le presse-papiers. |
| `android.permission.USE_BIOMETRIC` | Pour autoriser le déverrouillage biométrique optionnel, médié par Android Keystore. L'application n'accède pas directement aux données biométriques. |

Le workflow de release vérifie cet ensemble exact de permissions de la
plateforme avec `aapt2`. AndroidX ajoute également la permission
personnalisée
`com.brakovault.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`. Il
s'agit d'une permission privée, de niveau signature ; ce n'est pas une
permission de la plateforme Android et elle ne donne aucun accès au
réseau. La v0.3.0 pouvait afficher des permissions normales
supplémentaires ajoutées par AndroidX.

L'APK **n'a jamais déclaré et ne déclare toujours pas**
`android.permission.INTERNET`. Il n'y a pas de solution de repli, pas
d'exception « build debug uniquement » et aucun SDK tiers ne la
demande. Si une fonctionnalité future avait besoin d'un accès réseau,
elle serait reconsidérée ; la permission ne serait pas ajoutée.

## 10. Comment vérifier que l'APK n'a pas la permission INTERNET

Vous n'avez pas besoin de faire confiance à ce document. Vous pouvez
vérifier sur n'importe quel appareil ou poste de travail :

```shell
# Depuis Android SDK build-tools :
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# ou
aapt dump permissions brako-vault-vX.Y.Z.apk
```

Pour la v0.4.0 et les versions ultérieures, les permissions attendues de
la plateforme Android sont exactement les trois de la §9. `aapt2` peut
également afficher la permission privée d'AndroidX qui y est décrite. La
v0.3.0 pouvait afficher d'autres permissions normales ajoutées par
AndroidX. Si `android.permission.INTERNET` apparaît dans une version,
l'APK ne correspond pas à ce document ; ne l'installez pas.

## 11. Provenance et signature des binaires

Chaque release est signé avec la clé du mainteneur. À partir de v0.4.0,
il inclut aussi `SIGNING-CERTIFICATE.txt` et `SHA256SUMS.txt` ; les
versions antérieures ne les incluent pas. Pour inspecter localement :

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

L'empreinte SHA-256 officielle et canonique du certificat de signature
est :

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

À partir de la v0.4.0, le workflow de release exige cette empreinte.
L'empreinte SHA-256 affichée par `apksigner` doit correspondre à la fois
à la valeur ci-dessus et à `SIGNING-CERTIFICATE.txt`, et les hash des
artefacts doivent correspondre à `SHA256SUMS.txt`.

`SHA256SUMS.txt` et `SIGNING-CERTIFICATE.txt` ne disposent pas d'une
signature distincte. Ils permettent de détecter une corruption, mais ne
peuvent pas, à eux seuls, détecter la compromission d'un compte ou d'un
dépôt GitHub, puisqu'un attaquant pourrait les remplacer avec les
artefacts. La signature de l'APK et l'empreinte ancrée ci-dessus
authentifient l'APK ; elles n'authentifient pas un fichier AAB ou BLF.

## 12. Niveaux d'assurance

Les preuves disponibles ont des portées distinctes :

- **Déclaration de conception.** Ce document public décrit la conception
  et les paramètres exacts.
- **Tests internes.** Les tests privés vérifient des paramètres dans un
  document privé et le comportement interne. Ils ne prouvent pas
  l'examen de ces documents publics ni de l'APK/manifeste publié.
- **Vérification externe manuelle.** Chacun peut inspecter certificat et
  permissions avec les commandes ci-dessus et, dès v0.4.0, comparer les
  hash et l'empreinte aux deux fichiers de release ainsi qu'à
  l'empreinte canonique de ce document, dans les limites indiquées en
  §11.

Brako Vault **n'a pas** reçu d'audit externe, de certification,
d'évaluation Common Criteria ni de test d'intrusion tiers. Aucune
garantie de build reproductible n'est donnée.

## 13. Signalement d'une vulnérabilité

Pour un signalement sensible, utilisez le
[signalement privé](https://github.com/waar19/brako-vault-releases/security/advisories/new).
Ne publiez pas de détails sensibles dans une issue. Pour les questions
non sensibles, utilisez les [issues publiques](https://github.com/waar19/brako-vault-releases/issues).
Aucun délai fixe d'accusé de réception ou de correction n'est promis.
