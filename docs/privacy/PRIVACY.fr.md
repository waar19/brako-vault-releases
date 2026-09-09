# Extension de navigateur Brako Vault — Politique de confidentialité

[English](../../PRIVACY.md) | [Español](PRIVACY.es.md) | [Português](PRIVACY.pt.md) | **[Français](PRIVACY.fr.md)** | [Deutsch](PRIVACY.de.md) | [Italiano](PRIVACY.it.md) | [日本語](PRIVACY.ja.md) | [简体中文](PRIVACY.zh-CN.md)

Date d’entrée en vigueur : 9 septembre 2026

## Champ d’application

Cette politique couvre l’extension Brako Vault pour les navigateurs basés sur
Chromium et Firefox, ainsi que son hôte de messagerie native facultatif pour
Windows.

## Données traitées

Pour remplir son objectif unique — utiliser les identifiants d’un coffre local
chiffré — l’extension traite :

- les données d’authentification, notamment identifiants et mots de passe ;
- l’origine et l’URL de la page de connexion active ;
- l’envoi explicite d’un formulaire lors de la préparation d’une proposition
  d’enregistrement ou de mise à jour ;
- le dossier du coffre et le nom de l’appareil saisis dans les paramètres ;
- les entrées du coffre choisies par l’utilisateur.

L’extension ne lit que les champs liés à la connexion. Elle ne collecte ni
historique entre onglets, ni cookies, ni données financières ou de santé, ni
identifiants publicitaires, analyses, rapports de panne ou télémétrie.

## Utilisation des données

Les données servent uniquement à déverrouiller et rechercher le coffre local,
afficher les identifiants correspondants, remplir les champs choisis et
préparer un enregistrement ou une mise à jour soumis à confirmation. Brako
Vault n’envoie jamais automatiquement un formulaire de site.

## Messagerie native locale

L’extension échange ces données avec `com.brakovault.desktop_host`, un
programme Windows installé séparément par l’utilisateur. Firefox considère
Native Messaging comme une transmission hors du navigateur ; le manifeste
déclare donc les informations d’authentification, l’activité de navigation et
l’activité sur les sites.

L’hôte est local et n’envoie aucune donnée sur le réseau. Brako Vault, Google,
Mozilla et les tiers ne reçoivent ni coffre, ni identifiants, ni URL, ni nom
d’appareil, ni chemin de dossier, ni information d’utilisation.

## Stockage et protection

Le coffre reste dans le dossier choisi sous forme de fichier chiffré
`vault.bvda`. L’hôte conserve sa configuration dans
`%LOCALAPPDATA%\Brako Vault\config.json`. Si le déverrouillage rapide Windows
Hello est activé, un enregistrement chiffré est stocké dans
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json` ; il ne contient ni mot
de passe maître ni clé maître en clair.

Le coffre utilise AES-256-GCM et Argon2id. Windows Hello vérifie l’utilisateur
Windows local avant le déverrouillage rapide ; il ne remplace pas le chiffrement
du coffre et ne garantit pas une protection matérielle.

## Partage, vente et traitement distant

Aucune donnée n’est vendue, louée, partagée avec des tiers, utilisée pour la
publicité ou le crédit, ni traitée par un service distant. Le produit n’a ni
compte utilisateur ni synchronisation serveur.

## Conservation et suppression

Brako Vault ne peut ni accéder aux données ni les supprimer à distance :

- désactiver Windows Hello supprime `quick-unlock.json` ;
- désinstaller l’hôte supprime ses inscriptions et l’accès rapide, mais
  conserve volontairement `config.json` et le coffre ;
- l’utilisateur peut supprimer manuellement `config.json` et le coffre après
  avoir fermé l’hôte ;
- désinstaller l’extension supprime les données gérées par le navigateur.

Supprimer le coffre est irréversible sans une autre sauvegarde chiffrée.

## Modifications et contact

Toute modification importante sera publiée à cette même URL avec une date
actualisée. Pour toute question de confidentialité ou de sécurité, utilisez le
[signalement privé de vulnérabilité GitHub](https://github.com/waar19/brako-vault-releases/security/advisories/new).
