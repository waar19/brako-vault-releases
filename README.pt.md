# Brako Vault

[English](README.md) | [Español](README.es.md) | **[Português](README.pt.md)** | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Repositório oficial de downloads do Brako Vault para Android.

O Brako Vault é um gerenciador de senhas que funciona totalmente
offline:

- O APK não declara a permissão `INTERNET`.
- Não usa contas, telemetria nem sincronização com servidores.
- Protege o cofre com AES-256-GCM e uma chave derivada com Argon2id.
- A senha mestra não pode ser recuperada. Se você a perder, o cofre se
  vai junto.

## Baixar e instalar

1. Abra a seção [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Baixe o arquivo `.apk` da versão mais recente.
3. Instale-o em um dispositivo Android com API 29 (Android 10) ou
   superior.
4. Guarde sua senha mestra em um lugar seguro. Não existe mecanismo de
   recuperação.

> **Se você vem de uma build de desenvolvimento** (Android Studio
> executando `app` diretamente, ou um APK assinado com a chave debug do
> Android Studio), desinstale-a **antes** de instalar a build assinada
> de release. As assinaturas são diferentes e o Android bloqueará a
> atualização com `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (gerenciadores
> como Obtainium mostram o erro como "FailureConflict"). Passos:
>
> 1. Configurações → Aplicativos → Brako Vault → Desinstalar.
> 2. Instalar a build de release a partir daqui.
> 3. Se você tinha um cofre, reimporte o `.bvda` (ele não é copiado
>    entre assinaturas).

## APK vs AAB

- **APK** (Android Package): o arquivo que você instala em um
  dispositivo.
- **AAB** (Android App Bundle): o formato que o Google Play espera;
  contém o mesmo código dividido pela configuração do dispositivo. Não
  é possível fazer side-load de um `.aab`, por isso este repositório
  publica um `.apk` além do `.aab`.

## Verificar a assinatura

Você pode conferir a assinatura do APK com as ferramentas oficiais do
Android:

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

## Verificar o checksum SHA-256

Cada release publica um arquivo `SHA256SUMS.txt` com os resumos de cada
artefato. Para verificar no Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

No macOS / Linux:

```shell
sha256sum -c SHA256SUMS.txt
```

Os resumos publicados devem coincidir byte a byte com os do
`SHA256SUMS.txt`. Se não coincidirem, não instale o arquivo.

## Sobre os arquivos "Source code" gerados automaticamente

O GitHub gera automaticamente os links `Source code (zip)` e
`Source code (tar.gz)` em cada release. Esses arquivos são gerados a
partir **deste** repositório e contêm apenas o README, o aviso de
segurança e a configuração do repositório. **Não** contêm o código
fonte do aplicativo, que é privado. Trate-os como documentação, não
como código.

## Filtro de senhas vazadas (opcional)

A base offline opcional de senhas vazadas e seus checksums são
publicados em
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Código fonte

Este repositório distribui apenas binários oficiais. O código fonte do
Brako Vault não é público nem está incluído neste repositório.
