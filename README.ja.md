# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | **[日本語](README.ja.md)** | [简体中文](README.zh-CN.md)

Android 版 Brako Vault の公式ダウンロードリポジトリです。

Brako Vault は完全にオフラインで動作するパスワードマネージャーです:

- APK は `INTERNET` 権限を宣言していません。
- アカウント、テレメトリ、サーバーとの同期は一切使用しません。
- 保管庫は AES-256-GCM と Argon2id で導出した鍵で保護します。
- マスターパスワードを復元する手段はありません。生体認証によるロック
  解除をすでに有効にしていた場合、その認証が有効な間は引き続き保管庫を
  開けます。直ちに新しいパスワードで暗号化した `.bvda` をエクスポート
  してデータを救出してください。生体認証が機能しない、有効にして
  いなかった、またはエクスポート前に無効になった場合、保管庫には
  アクセスできなくなります。元のマスターパスワードが復元されるわけでは
  ありません。

## ダウンロードとインストール

1. [Releases](https://github.com/waar19/brako-vault-releases/releases)
   セクションを開きます。
2. 最新版の `brako-vault-vX.Y.Z.apk`、`SHA256SUMS.txt`、
   `SIGNING-CERTIFICATE.txt` だけをダウンロードします。
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
> 1. 開発版で、暗号化された `.bvda` バックアップをエクスポートします。
>    エクスポートファイルには必ずパスワードを入力します。マスター
>    パスワードでも、別のパスワードでも構いません。
> 2. ファイルをアプリのプライベートストレージ外に保存し、存在していて
>    空でないことを確認します。
> 3. 確認後にのみ、設定 → アプリ → Brako Vault →
>    アンインストールへ進みます。**アンインストールすると、アプリの
>    プライベートストレージ内のデータは削除されます。**
> 4. 本リポジトリのリリース版をインストールし、暗号化された `.bvda`
>    をインポートします。
>
> **エクスポートを確認できていない場合は、開発版をアンインストール
> しないでください。**

## APK と AAB の違い

- **APK** (Android Package): 端末にインストールするファイルです。
- **AAB** (Android App Bundle): Google Play が想定する形式です。
  同じコードを端末構成ごとに分割して含みます。`.aab` はサイドロード
  できないため、本リポジトリでは `.aab` と併せて `.apk` も公開して
  います。

## 署名を検証する (v0.4.0 以降)

v0.4.0 以降、各リリースには `SIGNING-CERTIFICATE.txt` が含まれます。
公式の Android ツールで APK を確認します:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

`apksigner` が表示する SHA-256 指紋は
`SIGNING-CERTIFICATE.txt` の指紋と一致する必要があります。公式かつ
正規の SHA-256 指紋は次のとおりです:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

v0.4.0 以降、`apksigner` の結果と `SIGNING-CERTIFICATE.txt` の両方が
この指紋と完全に一致する必要があります。署名が異なる場合、リリース
ワークフローは失敗します。それ以前のバージョンにはこのファイルは
ありません。

`SHA256SUMS.txt` と `SIGNING-CERTIFICATE.txt` はバイナリと一緒に公開
されますが、個別には署名されていません。チェックサムで検出できるのは
破損や不完全なダウンロードであり、GitHub の侵害ではありません。APK の
署名とここに固定された指紋を組み合わせることで APK を認証できますが、
この保証は AAB や BLF ファイルには適用されません。

## SHA-256 チェックサムを検証する (v0.4.0 以降)

v0.4.0 以降、各リリースには各成果物のダイジェストを記載した
`SHA256SUMS.txt` が付属します。それ以前のバージョンにはこのファイル
はありません。Windows (PowerShell) での検証方法:

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

macOS では、ダウンロードした APK だけを検証します:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | shasum -a 256 -c -
```

Linux では、ダウンロードした APK だけを検証します:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | sha256sum -c -
```

APK のダイジェストは `SHA256SUMS.txt` 内の該当する項目と 1 バイト
単位で一致しなければなりません。一致しない場合は APK をインストール
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
