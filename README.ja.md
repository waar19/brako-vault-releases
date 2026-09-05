# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | **[日本語](README.ja.md)** | [简体中文](README.zh-CN.md)

Android 版 Brako Vault の公式ダウンロードリポジトリです。

Brako Vault は完全にオフラインで動作するパスワードマネージャーです:

- APK は `INTERNET` 権限を宣言していません。
- アカウント、テレメトリ、サーバーとの同期は一切使用しません。
- 保管庫は AES-256-GCM と Argon2id で導出した鍵で保護します。
- マスターパスワードは復元できません。失うと保管庫も失われます。

## ダウンロードとインストール

1. [Releases](https://github.com/waar19/brako-vault-releases/releases)
   セクションを開きます。
2. 最新版の `.apk` ファイルをダウンロードします。
3. API 29 (Android 10) 以降の Android デバイスにインストールします。
4. マスターパスワードは安全な場所に保管してください。復元手段は
   存在しません。

> **開発版から移行する場合** (Android Studio で `app` を直接実行して
> いる場合、または Android Studio のデバッグ鍵で署名された APK の場合)
> は、リリース署名版をインストールする**前に**アンインストールして
> ください。署名が異なるため、Android は
> `INSTALL_FAILED_UPDATE_INCOMPATIBLE` でアップグレードを拒否します
> (Obtainium などの管理画面では "FailureConflict" と表示されます)。
> 手順:
>
> 1. 設定 → アプリ → Brako Vault → アンインストール。
> 2. このリポジトリからリリース版をインストール。
> 3. 保管庫がある場合は `.bvda` を再インポートしてください (署名間で
>    はコピーされません)。

## APK と AAB の違い

- **APK** (Android Package): 端末にインストールするファイルです。
- **AAB** (Android App Bundle): Google Play が想定する形式です。
  同じコードを端末構成ごとに分割して含みます。`.aab` はサイドロード
  できないため、本リポジトリでは `.aab` と併せて `.apk` も公開して
  います。

## 署名を検証する

公式の Android ツールで APK の署名を確認できます:

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

## SHA-256 チェックサムを検証する

各リリースには、各成果物のダイジェストを記載した `SHA256SUMS.txt`
が付属しています。Windows (PowerShell) での検証方法:

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

macOS / Linux の場合:

```shell
sha256sum -c SHA256SUMS.txt
```

公表されているダイジェストは `SHA256SUMS.txt` と 1 バイト単位で一致
しなければなりません。一致しない場合はそのファイルをインストール
しないでください。

## 自動生成された「Source code」アーカイブについて

GitHub は各リリースで `Source code (zip)` および
`Source code (tar.gz)` のリンクを自動的に生成します。これらの
アーカイブは**本リポジトリ**から生成され、README・セキュリティ通知・
リポジトリ設定のみを含みます。非公開のアプリケーションソースコード
は**含まれません**。コードではなくドキュメントとして扱って
ください。

## 漏洩パスワードフィルター (オプション)

漏洩パスワードのオフラインデータベース (任意) とそのチェックサムは
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases)
で公開されています。

## ソースコード

本リポジトリは公式バイナリのみを配布します。Brako Vault のソース
コードは公開されておらず、本リポジトリにも含まれていません。
