# Brako Vault 浏览器扩展 — 隐私政策

[English](../../PRIVACY.md) | [Español](PRIVACY.es.md) | [Português](PRIVACY.pt.md) | [Français](PRIVACY.fr.md) | [Deutsch](PRIVACY.de.md) | [Italiano](PRIVACY.it.md) | [日本語](PRIVACY.ja.md) | **[简体中文](PRIVACY.zh-CN.md)**

生效日期：2026 年 9 月 9 日

## 适用范围

本政策适用于 Chromium 系浏览器和 Firefox 的 Brako Vault 扩展，以及用户可选安装的
Windows 本机消息传递主机。

## 处理的数据

为了实现唯一目的——使用本地加密保险库中的凭据——扩展会处理：

- 包括用户名和密码在内的身份验证信息；
- 当前登录页面的来源和 URL；
- 在准备保存或更新建议时由用户明确提交的表单；
- 用户在“设置”中输入的保险库文件夹和设备名称；
- 用户选择的保险库条目。

扩展只读取与登录有关的字段。它不会收集跨标签页浏览历史、Cookie、支付或健康数据、
广告标识符、分析数据、崩溃报告或遥测数据。

## 数据用途

数据仅用于解锁和搜索本地保险库、显示匹配的凭据、填充用户选择的字段，以及准备需要
用户确认的保存或更新建议。Brako Vault 绝不会自动提交网站表单。

## 本机消息传递

扩展与用户另行安装的 Windows 程序 `com.brakovault.desktop_host` 交换上述数据。
Firefox 将 Native Messaging 视为浏览器外传输，因此 manifest 会声明身份验证信息、
浏览活动和网站活动。

主机只在本机运行，不会通过网络发送数据。Brako Vault、Google、Mozilla 和任何第三方
都不会收到保险库、凭据、URL、设备名称、文件夹路径或使用信息。

## 存储和保护

保险库以加密文件 `vault.bvda` 保留在用户选择的文件夹中。主机将配置保存在
`%LOCALAPPDATA%\Brako Vault\config.json`。启用 Windows Hello 快速解锁后，加密记录
保存在 `%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`；该记录不包含明文主密码
或主密钥。

保险库使用 AES-256-GCM 和 Argon2id。Windows Hello 会在快速解锁前验证本地 Windows
用户；它不会替代保险库加密，也不保证由硬件提供保护。

## 共享、出售和远程处理

我们不会出售、出租或与第三方共享数据，不会将数据用于广告或信贷决策，也不会交由
远程服务处理。本产品没有用户账户，也没有服务器同步。

## 保留和删除

Brako Vault 无法远程访问或删除数据：

- 关闭 Windows Hello 会删除 `quick-unlock.json`；
- 卸载主机会删除注册信息和快速解锁记录，但会有意保留 `config.json` 和保险库；
- 关闭主机后，用户可以手动删除 `config.json` 和保险库；
- 卸载扩展会删除由浏览器管理的扩展数据。

如果没有其他加密备份，删除保险库将无法恢复。

## 变更与联系

重大变更会在同一 URL 公布并更新生效日期。如有隐私或安全问题，请使用
[GitHub 私密漏洞报告](https://github.com/waar19/brako-vault-releases/security/advisories/new)。
