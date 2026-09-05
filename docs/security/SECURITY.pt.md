# Brako Vault — Segurança e transparência

[English](../SECURITY.md) | [Español](SECURITY.es.md) | **[Português](SECURITY.pt.md)** | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)**

Última atualização: 2026-09-05

Este documento descreve como o Brako Vault protege os dados no seu
dispositivo. Cada afirmação factual aqui é derivada do
[arquivo canônico de parâmetros criptográficos](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md)
público do repositório de código, ou pode ser verificada diretamente
no APK publicado e no manifesto do aplicativo. Onde uma afirmação é
"projetada" mas não "auditada", o documento o diz.

> **O código-fonte é privado.** O código do aplicativo não está neste
> repositório. Os números, tamanhos e protocolos abaixo são a
> declaração pública autoritativa do que o app faz. O APK de release é
> compilado a partir do mesmo código que contém as constantes em
> `Argon2Spec` e `BvdaConstants` no repositório privado.

## 1. Cifragem

| O quê | Como | Parâmetros |
|-------|------|------------|
| Cifra simétrica | AES-256-GCM (NIST SP 800-38D) | Chave de 256 bits, tag de autenticação de 128 bits, nonce de 96 bits (12 bytes) por escrita |
| Derivação de chave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iterações, 4 lanes, sal de 16 bytes, chave derivada de 32 bytes |
| Sal por cofre | Aleatório, 16 bytes | Gerado por uma fonte aleatória criptograficamente segura na criação do cofre |
| Nonce por escrita | Aleatório, 12 bytes | Único por escrita, nunca reutilizado com a mesma chave |
| Integridade do header | Ligada ao ciphertext | 46 bytes do header autenticados como Additional Authenticated Data (AAD) |

**O que significa "tag".** AES-GCM produz um tag de autenticação de
128 bits que é anexado ao ciphertext. Qualquer modificação do arquivo
(magic, parâmetros KDF, sal, nonce ou ciphertext) faz a verificação do
tag falhar e o cofre se recusa a abrir.

**O que significa "nonce".** O nonce de 12 bytes é um valor que nunca
deve se repetir sob a mesma chave. O app gera um nonce aleatório novo
em cada salvamento. Reutilizar um nonce seria catastrófico; por isso a
implementação falha explicitamente de forma segura se a fonte
aleatória estiver quebrada.

## 2. Formato de arquivo

Todos os dados do cofre são armazenados em um único arquivo:
`vault.bvda`.

Layout (big-endian para campos multi-byte, exceto `saltLen` e
`nonceLen` que são 1 byte):

```
 offset  tam  campo
 ------  ---  -----
   0      4   magic            = "BVDA" (ASCII)
   4      1   formatVersion    = 0x01
   5      1   kdfId            = 0x01 (Argon2id)
   6      1   aeadId           = 0x01 (AES-256-GCM)
   7      1   kdfParallelism   = 0x04
   8      4   kdfMemoryKiB     BE = 65 536
  12      4   kdfIterations    BE = 3
  16      1   saltLen          = 0x10 (16)
  17     16   kdfSalt
  33      1   nonceLen         = 0x0C (12)
  34     12   aeadNonce
  46      4   ciphertextLen    BE  <- NÃO faz parte do AAD
  50      N   ciphertext (N = ciphertextLen, inclui tag de 16 B)
```

Os primeiros 46 bytes do header são o AAD. O tag faz parte do
ciphertext, então o conteúdo autenticado do arquivo cobre tudo, exceto
o campo de 4 bytes com o comprimento do ciphertext.

## 3. Escrita atômica

Cada salvamento usa o padrão escrever-depois-renomear. O app:

1. Escreve o novo cofre em `vault.bvda.tmp`.
2. Chama `fsync` no arquivo temporário.
3. Chama `fsync` no diretório.
4. Renomeia atomicamente `vault.bvda.tmp` para `vault.bvda`.

Se o dispositivo perder energia ou o app for morto entre as etapas, o
`vault.bvda` anterior fica intacto e o arquivo temporário vira lixo
(limpo no próximo salvamento bem-sucedido).

## 4. Proteção biométrica da chave

Quando o desbloqueio biométrico está ativado, a chave derivada é
embrulhada com uma chave guardada no Android Keystore. A chave
protegida pelo Keystore:

- Nunca sai do hardware seguro quando o dispositivo tem um Trusted
  Execution Environment (TEE) ou um StrongBox Keymaster.
- Não pode ser extraída por processos em modo usuário.
- É invalidada quando o usuário remove todas as biometrias do
  dispositivo, troca a tela de bloqueio, ou faz reset de fábrica.

O prompt biométrico é imposto pelo sistema operacional; o app não pode
burlá-lo.

## 5. Sem recuperação de senha

Não há mecanismo de recuperação, nem redefinição por e-mail, nem chave
de recuperação, nem atendimento ao cliente que possa abrir um cofre. A
senha mestra é a única chave, nunca é armazenada, nunca é transmitida,
nunca é registrada. Se você perder a senha mestra, o cofre se vai
junto.

O autor deste projeto também não pode abrir o seu cofre.

## 6. Modelo de ameaças — o que o Brako Vault protege

O app é projetado para proteger contra:

- Roubo do dispositivo desligado, por um adversário sem a senha mestra
  e sem capacidade de contornar a criptografia de disco do Android.
- Ataques pela rede: o app não tem permissão `INTERNET`, então
  condições de rede comprometidas não podem exfiltrar o cofre.
- Replay ou modificação do arquivo do cofre: qualquer mudança de um
  bit é detectada pela verificação do tag AES-GCM.
- Força bruta sobre a senha mestra: Argon2id com 64 MiB e 3
  iterações torna cada tentativa custosa; um atacante offline ainda
  precisa adivinhar a senha.

O app **não** é projetado para proteger contra:

- Um dispositivo comprometido ou rooteado rodando enquanto o cofre está
  aberto. Depois que a senha mestra é verificada, a chave está na
  memória e o processo tem acesso ao plaintext.
- Olhares de terceiros, keyloggers ou malware rodando com o mesmo UID
  do app.
- Coerção: um atacante determinado com acesso físico a um dispositivo
  desbloqueado pode ler o cofre.
- Uma senha mestra fraca ou reutilizada. Argon2id torna o ataque mais
  lento, mas não torna "123456" seguro.
- O usuário que exporta voluntariamente o cofre sem cifragem para um
  terceiro.

## 7. Backups

O app exporta um arquivo `.bvda` (o mesmo formato cifrado do cofre em
disco). O arquivo exportado é cifrado com a mesma chave derivada da
senha mestra, a menos que o usuário escolha explicitamente "exportar
sem senha" — e nesse caso o app avisa em voz alta de que o arquivo
fica desprotegido. Por padrão, o sistema de backup do dispositivo é
desativado para manter o cofre fora dos arquivos de `adb backup` e dos
backups na nuvem.

## 8. Sincronização offline

O Brako Vault não tem servidor. A sincronização entre dispositivos
funciona trocando arquivos `.bvda` por um canal que o usuário escolhe
(USB, e-mail, armazenamento na nuvem, AirDrop). O app nunca abre um
socket de rede, então o canal não pode ser observado pelo próprio app.
O arquivo é autenticado de ponta a ponta pela mesma verificação do tag
AES-GCM, então um terceiro que retransmite uma cópia adulterada é
detectado na importação.

## 9. Permissões declaradas

O APK declara exatamente três permissões:

| Permissão | Por quê |
|-----------|---------|
| `android.permission.CAMERA` | Para escanear QR codes com tokens 2FA. A câmera é usada apenas quando o usuário abre o scanner de QR; os quadros são processados no dispositivo e nunca são guardados. |
| `android.permission.VIBRATE` | Para dar feedback háptico em ações como desbloqueio bem-sucedido ou copiar para a área de transferência. |
| `android.permission.USE_BIOMETRIC` | Para permitir desbloqueio biométrico opcional, mediado pelo Android Keystore. O app não acessa dados biométricos diretamente. |

O APK **não** declara `android.permission.INTERNET`. Não há fallback,
não há exceção de "apenas build debug", nem SDK de terceiros que
solicite. Se uma feature futura precisar de rede, a feature é
reconsiderada; a permissão não é adicionada.

## 10. Como verificar que o APK não tem permissão INTERNET

Você não precisa confiar neste documento. Você pode verificar em
qualquer dispositivo ou estação de trabalho:

```shell
# Do Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# ou
aapt dump permissions brako-vault-vX.Y.Z.apk
```

A saída deve listar apenas as três permissões em §9. Se
`android.permission.INTERNET` aparecer, o arquivo não é o APK oficial
— não instale.

## 11. Procedência e assinatura dos binários

Cada release é assinado com a chave de release do mantenedor. A
impressão digital é publicada nas notas do release. Para verificar
localmente:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

A impressão digital SHA-256 do certificado de assinatura deve
corresponder à das notas do release. Os checksums de cada artefato
estão em `SHA256SUMS.txt`, ao lado dos binários do mesmo release.

## 12. Níveis de garantia

O Brako Vault é oferecido com três níveis de garantia explícitos. Eles
não são a mesma coisa:

- **Projetado.** A arquitetura e as escolhas de parâmetros deste
  documento são o que o autor se comprometeu a implementar. É a
  afirmação mais fraca.
- **Testado automaticamente.** Uma bateria de testes no repositório
  privado de código — `CryptoSpecDocTest` e o restante de `jvmTest` e
  `androidApp:testDebugUnitTest` — roda a cada push e verifica que a
  implementação corresponde ao declarado aqui, incluindo os parâmetros
  criptográficos e a ausência de `INTERNET` no manifesto.
- **Auditado externamente.** O Brako Vault **não** foi auditado por
  uma parte externa. O autor não faz afirmações de certificação
  externa, avaliação de common criteria nem pentest de terceiros. Se
  uma auditoria futura for feita, seus achados serão publicados aqui
  com data, escopo e o relatório completo.

## 13. Reporte de vulnerabilidades

Se você encontrar uma vulnerabilidade, escreva para
`security@brakovault.example` (substitua pelo endereço real quando o
projeto o tornar público). Não abra uma issue pública no GitHub para
relatos sensíveis de segurança. O autor se compromete a confirmar
recebimento em 72 horas e a fornecer uma correção ou uma aceitação
documentada do risco em 30 dias para problemas confirmados.
