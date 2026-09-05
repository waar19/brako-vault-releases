# Brako Vault — 安全与透明

[English](../../SECURITY.md) | [Español](SECURITY.es.md) | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | **[简体中文](SECURITY.zh-CN.md)**

最后更新:2026-09-05

本文档是 Brako Vault 独立完整的公开安全与透明度声明。源代码仓库
为私有;本公开 release 仓库不提供源代码,也不声称第三方可复现构建。
APK 的签名和权限可手动核验。保证范围见 §12。

## 1. 加密

| 项目 | 方式 | 参数 |
|------|------|------|
| 对称密码 | AES-256-GCM (NIST SP 800-38D) | 32 字节 (256 位)密钥、16 字节 (128 位)标签、12 字节 (96 位)nonce |
| 密钥派生 | Argon2id (RFC 9106) | 64 MiB (65 536 KiB)、3 轮迭代、并行度 4、16 字节 salt、32 字节密钥 |
| 每库 salt | 随机 16 字节 | 在创建时由密码学安全随机源生成 |
| 每次写入 nonce | 随机 12 字节 | 每次向 Java `SecureRandom` 请求 12 个新字节 |
| 头部 | 50 字节 | 前 46 字节为 AAD;最后 4 字节 `ciphertextLen` 未认证 |

**何为"标签"。** AES-GCM 的 16 字节标签认证头部前 46 字节(AAD)
及声明的密文。`ciphertextLen` 未认证。当前读取器允许并忽略声明
密文之后的字节;标签不覆盖这些字节。

**何为"nonce"。** 12 字节 nonce 是在同一密钥下绝不可重复的值。
应用每次向 Java `SecureRandom` 请求 12 个新字节。实现没有明确的
重复检测器,也不检查随机源故障;这是概率性防护,不是绝对唯一或
fail-closed 保证。

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

认证区域包括头部前 46 字节、声明密文及 GCM 标签。`ciphertextLen`
未认证,后续字节被允许并忽略。

## 3. 写入替换与持久性限制

每次保存都遵循"先写后改名"模式。应用:

1. 将新的保险库写入 `vault.bvda.tmp`。
2. 同步临时文件描述符。
3. 尝试带替换的 `ATOMIC_MOVE`。
4. 失败后尝试非原子 move,再失败则 copy+delete。

目录不会同步。仅第一种方式在文件系统支持时力求原子性;回退方式
不保证原子替换。故障可能留下不完整或缺失的目标文件,不承诺完全的
崩溃持久性。

## 4. 生物识别密钥保护

派生密钥由 Android Keystore 密钥包裹。该密钥无法通过 Android API
导出,要求 `BIOMETRIC_STRONG`,并在重新录入生物特征时失效。

硬件、TEE 或 StrongBox 支持取决于设备。应用不请求 StrongBox,也不
验证硬件支持;不能假定所有设备都具备这些属性。

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
- 已认证 BVDA 内容的篡改:可检测头部前 46 字节、声明密文或 GCM
  标签的变化。
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
- 重放或回滚到较旧的有效 `.bvda`;AES-GCM 不证明新鲜度或版本单调性。
- 未认证的 `ciphertextLen` 或后续字节变化。
- 泄露导出密码或使用不可信传输渠道。

## 7. 备份

每次导出 `.bvda` 都要求用户输入密码,可以是主密码或其他密码。
不存在无密码导出。设备备份默认关闭。

## 8. 离线同步

Brako Vault 没有服务器。设备之间的同步通过用户选定的通道
(USB、邮件、云存储、类 AirDrop 共享)交换 `.bvda` 文件完成。
应用从不打开网络套接字。导入时 AES-GCM 校验已认证区域,但无法
检测旧有效文件的重放或 §2 所述未认证数据。

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

预期输出仅含 §9 的三项权限。若出现 `android.permission.INTERNET`,
该 APK 与本文声明不符;请勿安装。

## 11. 二进制来源与签名

每个 release 均由维护者密钥签名。从 v0.4.0 起还包含
`SIGNING-CERTIFICATE.txt` 与 `SHA256SUMS.txt`;此前版本没有。
本地验证:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

从 v0.4.0 起,`apksigner` 的 SHA-256 指纹须与
`SIGNING-CERTIFICATE.txt` 一致,产物哈希须与 `SHA256SUMS.txt` 一致。

## 12. 保证等级

现有证据范围不同:

- **设计声明。** 本公开文档说明设计和精确参数。
- **内部测试。** 私有测试验证私有文档中的规范参数和内部行为;不能
  证明本公开文档或已发布 APK/manifest 已受检查。
- **外部手动验证。** 任何人可用上述命令检查证书和权限;从 v0.4.0
  起还可比较哈希与指纹。

Brako Vault **未** 接受外部安全审计、认证、Common Criteria 评估或
第三方渗透测试,也不保证构建可复现。

## 13. 报告漏洞

敏感报告请使用
[私有漏洞报告](https://github.com/waar19/brako-vault-releases/security/advisories/new)。
请勿在公开 issue 中写入敏感细节。非敏感问题可使用
[公开 issues](https://github.com/waar19/brako-vault-releases/issues)。
不承诺固定的确认或修复期限。
