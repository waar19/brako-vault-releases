# Extensão de navegador Brako Vault — Política de privacidade

[English](../../PRIVACY.md) | [Español](PRIVACY.es.md) | **[Português](PRIVACY.pt.md)** | [Français](PRIVACY.fr.md) | [Deutsch](PRIVACY.de.md) | [Italiano](PRIVACY.it.md) | [日本語](PRIVACY.ja.md) | [简体中文](PRIVACY.zh-CN.md)

Data de vigência: 9 de setembro de 2026

## Âmbito

Esta política abrange a extensão Brako Vault para navegadores baseados em
Chromium e Firefox e o host opcional de mensagens nativas para Windows.

## Dados tratados

Para cumprir sua única finalidade — usar credenciais de um cofre local
criptografado — a extensão trata:

- dados de autenticação, incluindo nomes de usuário e senhas;
- a origem e a URL da página de login ativa;
- o envio explícito de um formulário ao preparar uma proposta de salvamento
  ou atualização;
- a pasta do cofre e o nome do dispositivo informados nas Configurações;
- as entradas do cofre selecionadas pelo usuário.

A extensão só lê campos relacionados ao login. Não coleta histórico entre
abas, cookies, dados financeiros ou de saúde, identificadores publicitários,
análises, relatórios de falhas ou telemetria.

## Uso dos dados

Os dados são usados apenas para desbloquear e pesquisar o cofre local, mostrar
credenciais correspondentes, preencher os campos escolhidos pelo usuário e
preparar um salvamento ou atualização que exige confirmação. O Brako Vault
nunca envia automaticamente um formulário do site.

## Mensagens nativas locais

A extensão troca os dados acima com `com.brakovault.desktop_host`, um programa
Windows instalado separadamente pelo usuário. O Firefox classifica Native
Messaging como transmissão para fora do navegador; por isso o manifesto
declara informações de autenticação, atividade de navegação e atividade em
sites.

O host é local e não envia dados pela rede. Brako Vault, Google, Mozilla e
terceiros não recebem o cofre, credenciais, URLs, nome do dispositivo, caminho
da pasta ou informações de uso.

## Armazenamento e proteção

O cofre permanece na pasta escolhida pelo usuário como `vault.bvda`
criptografado. O host salva a configuração em
`%LOCALAPPDATA%\Brako Vault\config.json`. Se o desbloqueio rápido com Windows
Hello estiver ativo, salva um registro criptografado em
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`; ele não contém a senha
mestra nem a chave mestra em texto simples.

O cofre usa AES-256-GCM e Argon2id. O Windows Hello verifica o usuário local
antes do desbloqueio rápido; não substitui a criptografia do cofre nem garante
proteção baseada em hardware.

## Compartilhamento, venda e processamento remoto

Nenhum dado é vendido, alugado, compartilhado com terceiros, usado para
publicidade ou decisões de crédito, nem processado por serviço remoto. O
produto não possui contas nem sincronização com servidor.

## Retenção e exclusão

O Brako Vault não acessa nem exclui dados remotamente:

- desativar o Windows Hello remove `quick-unlock.json`;
- desinstalar o host remove registros e o acesso rápido, mas preserva
  intencionalmente `config.json` e o cofre;
- o usuário pode excluir manualmente `config.json` e o cofre após fechar o
  host;
- desinstalar a extensão remove os dados gerenciados pelo navegador.

Excluir o cofre é irreversível sem outro backup criptografado.

## Alterações e contato

Alterações relevantes serão publicadas nesta mesma URL e a data será
atualizada. Para questões de privacidade ou segurança, use o
[relato privado de vulnerabilidade do GitHub](https://github.com/waar19/brako-vault-releases/security/advisories/new).
