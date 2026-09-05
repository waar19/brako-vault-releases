# Brako Vault — 安全与透明

[English](../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | **[简体中文](SECURITY.zh-CN.md)**

最后更新:2026-09-05

本文档说明 Brako Vault 如何保护您设备上的数据。文中每一个事实性
陈述,要么来自源代码仓库中公开的
[权威加密参数文件](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md),
要么可以直接通过已发布的 APK 和应用清单进行验证。凡是"已设计"但
"未经审计"的陈述,文档中都会明确指出。

> **源代码为私有。** 本仓库不包含应用源代码。下文的数字、尺寸与
> 协议,是对应用行为的公开权威声明。release APK 由同一份代码编译
> 而成,该代码在私有仓库中包含了 `Argon2Spec` 与 `BvdaConstants`
> 中的常量。

## 1. 加密

| 项目 | 方式 | 参数 |
|------|------|------|
| 对称密码 | AES-256-GCM (NIST SP 800-38D) | 256 位密钥、128 位认证标签、每次写入 96 位 (12 字节) nonce |
| 密钥派生 | Argon2id (RFC 9106) | 64 MiB (65 536 KiB)、3 轮迭代、4 lanes、16 字节 salt、32 字节派生密钥 |
| 每库 salt | 随机 16 字节 | 在创建时由密码学安全随机源生成 |
| 每次写入 nonce | 随机 12 字节 | 每次写入唯一,绝不与同一密钥重复使用 |
| 头部完整性 | 与密文绑定 | 头部前 46 字节作为 Additional Authenticated Data (AAD) 进行认证 |

**何为"标签"。** AES-GCM 生成一个 128 位的认证标签,附加在密文
之后。对文件的任何修改(magic、KDF 参数、salt、nonce 或密文)都会
导致标签校验失败,保险库拒绝打开。

**何为"nonce"。** 12 字节 nonce 是在同一密钥下绝不可重复的值。
应用在每次保存时生成一个新的随机 nonce。重用 nonce 后果是灾难性
的;因此,一旦随机源失效,实现会以安全方式显式失败。

## 2. 文件格式

所有保险库数据存储在单一文件 `vault.bvda` 中。

布局(多字节字段采用大端序,`saltLen` 与 `nonceLen` 为 1 字节):

```
 偏移  大小  字段
 ----  ----  ----
   0     4   magic            = "BVDA" (ASCII)
   4     1   formatVersion    = 0x01
   5     1   kdfId            = 0x01 (Argon2id)
   6     1   aeadId           = 0x01 (AES-256-GCM)
   7     1   kdfParallelism   = 0x04
   8     4   kdfMemoryKiB     BE = 65 536
  12     4   kdfIterations    BE = 3
  16     1   saltLen          = 0x10 (16)
  17    16   kdfSalt
  33     1   nonceLen         = 0x0C (12)
  34    12   aeadNonce
  46     4   ciphertextLen    BE  <- 不属于 AAD
  50     N   ciphertext (N = ciphertextLen,末尾含 16 字节标签)
```

头部的前 46 字节即为 AAD。标签属于密文的一部分,因此文件的认证
范围覆盖除密文长度字段(4 字节)之外的全部内容。

## 3. 原子写入

每次保存都遵循"先写后改名"模式。应用:

1. 将新的保险库写入 `vault.bvda.tmp`。
2. 对临时文件调用 `fsync`。
3. 对目录调用 `fsync`。
4. 将 `vault.bvda.tmp` 原子地重命名为 `vault.bvda`。

若设备在上述步骤之间断电或应用被终止,既有的 `vault.bvda` 保持
不变,临时文件作为垃圾保留(在下一次成功保存时被清理)。

## 4. 生物识别密钥保护

当启用生物识别解锁时,派生密钥会被一个保存在 Android Keystore
中的密钥包裹。Keystore 保护的密钥具备以下特性:

- 在设备具备 Trusted Execution Environment (TEE) 或 StrongBox
  Keymaster 时,永不离开安全硬件。
- 用户态进程不可导出。
- 当用户移除设备的全部生物特征、更改锁屏,或恢复出厂设置时,
  密钥即失效。

生物识别的提示由操作系统强制执行;应用无法绕过。

## 5. 无法找回密码

不存在任何找回机制,没有邮箱重置,没有恢复密钥,也没有客服可以
打开保险库。主密码是唯一的密钥,既不被存储,也不被传输,更不会
写入日志。丢失主密码,保险库即丢失。

本项目作者也无法打开您的保险库。

## 6. 威胁模型 — Brako Vault 保护的范围

本应用设计用于抵御:

- 设备在关机状态被盗:攻击者既无主密码,也没有绕过 Android
  磁盘加密的能力。
- 网络攻击:应用不具有 `INTERNET` 权限,即使网络被攻陷也无法
  外泄保险库。
- 保险库文件的重放或篡改:AES-GCM 标签校验可检测任何位翻转。
- 对主密码的暴力破解:64 MiB、3 轮迭代的 Argon2id 让每次猜测
  都代价高昂;离线攻击者仍需猜测密码本身。

本应用 **不** 旨在抵御:

- 设备被入侵或 root 后,在保险库处于打开状态时运行。一旦主密码
  验证通过,密钥即在内存中,进程可访问明文。
- 偷窥、键盘记录器,或与本应用同 UID 运行的恶意软件。
- 胁迫:决心坚定的攻击者若物理接触到一台已解锁的设备,可以
  读取保险库。
- 弱主密码或重复使用的主密码。Argon2id 减缓破解速度,但不能让
  "123456" 变得安全。
- 用户主动将未加密的保险库导出给第三方。

## 7. 备份

应用可导出 `.bvda` 文件(与磁盘上保险库相同的加密格式)。导出的
文件使用与主密码派生的同一密钥加密,除非用户明确选择"无密码
导出" — 在这种情况下,应用会大声警告该文件将失去保护。设备自带
的备份系统默认被关闭,以避免保险库进入 `adb backup` 归档或云端
备份。

## 8. 离线同步

Brako Vault 没有服务器。设备之间的同步通过用户选定的通道
(USB、邮件、云存储、类 AirDrop 共享)交换 `.bvda` 文件完成。
应用从不打开网络套接字,因此通道本身不能被应用所观察。文件由
同一套 AES-GCM 标签校验进行端到端认证,因此中转被篡改副本的第
三方会在导入时被检测出来。

## 9. 声明的权限

APK 仅声明以下三项权限:

| 权限 | 用途 |
|------|------|
| `android.permission.CAMERA` | 用于扫描包含 2FA 令牌的二维码。摄像头仅在用户打开二维码扫描器时使用;画面在设备上处理,从不保存。 |
| `android.permission.VIBRATE` | 用于在解锁成功、复制到剪贴板等操作时提供触觉反馈。 |
| `android.permission.USE_BIOMETRIC` | 用于在 Android Keystore 的中介下,允许可选的生物识别解锁。应用不直接访问生物数据。 |

APK **不** 声明 `android.permission.INTERNET`。没有回退路径,
没有"仅 debug 构建"的例外,也没有第三方 SDK 索要它。若未来某项
功能需要联网,会重新审视该功能本身,而不会增加权限。

## 10. 如何验证 APK 不含 INTERNET 权限

您不必轻信本文档。可在任意设备或工作站上验证:

```shell
# 通过 Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# 或
aapt dump permissions brako-vault-vX.Y.Z.apk
```

输出必须仅列出 §9 中的三项权限。若出现
`android.permission.INTERNET`,则该文件并非官方 APK — 请勿安装。

## 11. 二进制来源与签名

每个 release 均由维护者的 release 密钥签名。指纹在 release 说明
中公开。本地验证方式:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

签名证书的 SHA-256 指纹必须与 release 说明中的一致。每个产物的
校验和位于同一 release 的二进制旁边的 `SHA256SUMS.txt` 中。

## 12. 保证等级

Brako Vault 以三个明确的保证等级提供。它们不是同一件事:

- **已设计 (Designed)。** 本文档中的架构与参数选择,是作者承诺
  实现的内容。这是最弱的声明。
- **自动测试通过 (Automatically tested)。** 私有源代码仓库中的
  测试套件 — `CryptoSpecDocTest`、`jvmTest`、
  `androidApp:testDebugUnitTest` — 在每次 push 时运行,验证实现
  与本文档的声明一致,包括加密参数以及清单中不包含 `INTERNET`。
- **外部审计 (Externally audited)。** Brako Vault **未** 经过
  外部审计。作者不主张任何外部认证、Common Criteria 评估或
  第三方渗透测试。若未来完成审计,其结论将连同日期、范围与
  完整报告一并在此发布。

## 13. 报告漏洞

如发现漏洞,请发送邮件至 `security@brakovault.example`(项目
公开真实地址后请替换)。请勿就安全敏感的报告提交公开的
GitHub issue。作者承诺在 72 小时内确认收到,并在 30 天内为
已确认的问题提供修复或书面的风险接受。
