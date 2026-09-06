# Brako Vault

[English](README.md) | [Español](README.es.md) | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | **[简体中文](README.zh-CN.md)**

Brako Vault Android 版官方下载仓库。

Brako Vault 是一款完全离线工作的密码管理器:

- APK 不声明 `INTERNET` 权限。
- 不使用账号、不发送遥测、不与任何服务器同步。
- 使用 AES-256-GCM 与 Argon2id 派生的密钥保护保险库。
- 主密码无法找回。如果此前已启用生物识别解锁,它仍然有效,可继续打开
  保险库。请立即使用新密码导出加密的 `.bvda`,以便挽救数据。如果生物
  识别无法使用、此前未启用或在导出前失效,保险库将无法访问。这并不能
  找回原主密码。

## 下载与安装

1. 打开 [Releases](https://github.com/waar19/brako-vault-releases/releases)
   页面。
2. 仅下载最新版本的 `brako-vault-vX.Y.Z.apk`、`SHA256SUMS.txt` 和
   `SIGNING-CERTIFICATE.txt`。
3. 在 API 29 (Android 10) 或更高版本的 Android 设备上安装。
4. 请将主密码妥善保管。系统不提供任何找回机制。

> **如果你是从开发版本升级过来** (在 Android Studio 中直接运行
> `app`,或使用 Android Studio 的 debug 密钥签名的 APK),请在安装
> release 签名版本之前**先卸载**它。两者的签名不同,Android 会
> 以 `INSTALL_FAILED_UPDATE_INCOMPATIBLE` 阻止升级 (Obtainium 等
> 安装管理器会显示 "FailureConflict")。步骤:
>
> 1. 在开发版本中导出一份加密的 `.bvda` 备份。导出文件必须输入
>    密码;可以使用主密码,也可以使用其他密码。
> 2. 将文件保存到应用私有存储之外,并确认文件存在且不为空。
> 3. 只有完成确认后,才可前往设置 → 应用 → Brako Vault → 卸载。
>    **卸载会删除应用私有存储中的数据。**
> 4. 从本仓库安装 release 版本,然后导入加密的 `.bvda`。
>
> **如果没有确认导出成功,请勿卸载开发版本。**

## APK 与 AAB 的区别

- **APK** (Android Package): 直接安装到设备上的文件。
- **AAB** (Android App Bundle): Google Play 所要求的格式;包含相同
  代码,但按设备配置拆分。`.aab` 不能直接侧载(side-load),因此本
  仓库除 `.aab` 之外还发布一份 `.apk`。

## 验证签名 (v0.4.0 及以后版本)

从 v0.4.0 开始,每个 release 都会发布
`SIGNING-CERTIFICATE.txt`。使用 Android 官方工具校验 APK:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

`apksigner` 显示的 SHA-256 指纹必须与
`SIGNING-CERTIFICATE.txt` 中的指纹一致。官方且规范的 SHA-256 指纹为:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

从 v0.4.0 开始,`apksigner` 的结果和 `SIGNING-CERTIFICATE.txt` 都必须
与此指纹完全一致;如果签名不同,发布工作流将失败。此前版本不包含
此文件。

`SHA256SUMS.txt` 和 `SIGNING-CERTIFICATE.txt` 与二进制文件一同发布,
没有单独的签名。校验和可以检测文件损坏或下载不完整,但无法检测 GitHub
遭到入侵。APK 签名与此处固定的指纹共同验证 APK 的真实性;此保证不适用于
AAB 或 BLF 文件。

## 验证 SHA-256 校验和 (v0.4.0 及以后版本)

从 v0.4.0 开始,每个 release 都会发布 `SHA256SUMS.txt`,列出每个
产物的摘要。此前版本不包含此文件。在 Windows (PowerShell) 中验证:

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

在 macOS 中,仅验证下载的 APK:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | shasum -a 256 -c -
```

在 Linux 中,仅验证下载的 APK:

```shell
awk '$2 == "brako-vault-vX.Y.Z.apk" {print}' SHA256SUMS.txt | sha256sum -c -
```

APK 摘要必须与 `SHA256SUMS.txt` 中对应的条目逐字节一致。如果不一致,
请勿安装该 APK。

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
