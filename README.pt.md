# Brako Vault

[English](README.md) | [Español](README.es.md) | **[Português](README.pt.md)** | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Repositório oficial de downloads do Brako Vault para Android.

O Brako Vault é um gerenciador de senhas que funciona totalmente
offline:

- O APK não declara a permissão `INTERNET`.
- Não usa contas, telemetria nem sincronização com servidores.
- Protege o cofre com AES-256-GCM e uma chave derivada com Argon2id.
- A senha mestra não pode ser recuperada. Se o desbloqueio biométrico já
  estava habilitado, ele continua válido e ainda permite abrir o cofre:
  exporte imediatamente um `.bvda` cifrado com uma nova senha para resgatar
  seus dados. Se a biometria não funcionar, não tiver sido habilitada ou for
  invalidada antes da exportação, o cofre ficará inacessível. Isso não
  recupera a senha mestra original.

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
> 1. Na build de desenvolvimento, exporte uma cópia `.bvda` cifrada.
>    Você deve informar uma senha para o arquivo exportado; pode ser a
>    senha mestra ou outra.
> 2. Salve o arquivo fora do armazenamento privado do app e confirme
>    que ele existe e não está vazio.
> 3. Só então vá a Configurações → Aplicativos → Brako Vault →
>    Desinstalar. **A desinstalação apaga os dados no armazenamento
>    privado do app.**
> 4. Instale a build de release deste repositório e importe o `.bvda`
>    cifrado.
>
> **Se você não tiver uma exportação confirmada, não desinstale a build
> de desenvolvimento.**

## APK vs AAB

- **APK** (Android Package): o arquivo que você instala em um
  dispositivo.
- **AAB** (Android App Bundle): o formato que o Google Play espera;
  contém o mesmo código dividido pela configuração do dispositivo. Não
  é possível fazer side-load de um `.aab`, por isso este repositório
  publica um `.apk` além do `.aab`.

## Verificar a assinatura (v0.4.0 e posteriores)

A partir da v0.4.0, cada release publica
`SIGNING-CERTIFICATE.txt`. Confira o APK com as ferramentas oficiais do
Android:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

A impressão digital SHA-256 exibida pelo `apksigner` deve corresponder
à de `SIGNING-CERTIFICATE.txt`. Versões anteriores não incluem esse
arquivo.

## Verificar o checksum SHA-256 (v0.4.0 e posteriores)

A partir da v0.4.0, cada release publica `SHA256SUMS.txt` com os
resumos de cada artefato. Versões anteriores não incluem esse arquivo.
Para verificar no Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

No macOS:

```shell
shasum -a 256 -c SHA256SUMS.txt
```

No Linux:

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
