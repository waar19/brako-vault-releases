# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | **[简体中文](README.zh-CN.md)**

Brako Vault Android 版官方下载仓库。

Brako Vault 是一款完全离线工作的密码管理器:

- APK 不声明 `INTERNET` 权限。
- 不使用账号、不发送遥测、不与任何服务器同步。
- 使用 AES-256-GCM 与 Argon2id 派生的密钥保护保险库。
- 主密码无法找回。一旦丢失,保险库即随之消失。

## 下载与安装

1. 打开 [Releases](https://github.com/waar19/brako-vault-releases/releases)
   页面。
2. 下载最新版本的 `.apk` 文件。
3. 在 API 29 (Android 10) 或更高版本的 Android 设备上安装。
4. 请将主密码妥善保管。系统不提供任何找回机制。

> **如果你是从开发版本升级过来** (在 Android Studio 中直接运行
> `app`,或使用 Android Studio 的 debug 密钥签名的 APK),请在安装
> release 签名版本之前**先卸载**它。两者的签名不同,Android 会
> 以 `INSTALL_FAILED_UPDATE_INCOMPATIBLE` 阻止升级 (Obtainium 等
> 安装管理器会显示 "FailureConflict")。步骤:
>
> 1. 设置 → 应用 → Brako Vault → 卸载。
> 2. 从本仓库安装 release 版本。
> 3. 如果你之前有保险库,请重新导入 `.bvda` (不同签名之间不会
>    复制)。

## APK 与 AAB 的区别

- **APK** (Android Package): 直接安装到设备上的文件。
- **AAB** (Android App Bundle): Google Play 所要求的格式;包含相同
  代码,但按设备配置拆分。`.aab` 不能直接侧载(side-load),因此本
  仓库除 `.aab` 之外还发布一份 `.apk`。

## 验证签名

可使用 Android 官方工具校验 APK 签名:

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

## 验证 SHA-256 校验和

每个 release 都附带一份 `SHA256SUMS.txt`,列出每个产物的摘要。在
Windows (PowerShell) 中验证:

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

在 macOS / Linux 中:

```shell
sha256sum -c SHA256SUMS.txt
```

发布的摘要必须与 `SHA256SUMS.txt` 中的逐字节一致。如果不一致,请
不要安装该文件。

## 关于自动生成的 "Source code" 压缩包

GitHub 会为每个 release 自动生成 `Source code (zip)` 与
`Source code (tar.gz)` 链接。这些压缩包由**本仓库**生成,仅包含
README、安全说明以及仓库配置,**不包含**应用源代码(源代码为
私有)。请将它们视作文档,而非代码。

## 泄漏密码过滤器(可选)

可选的离线泄漏密码数据库及其校验和发布于
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases)。

## 源代码

本仓库仅分发官方二进制文件。Brako Vault 的源代码不公开,也不包含
在本仓库中。
