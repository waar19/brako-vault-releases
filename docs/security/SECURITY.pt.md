# Brako Vault — Segurança e transparência

[English](../../SECURITY.md) | [Español](SECURITY.es.md) | **[Português](SECURITY.pt.md)** | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)

Última atualização: 2026-09-05

Este documento autocontido é a declaração pública de segurança e
transparência do Brako Vault. O repositório do código do aplicativo é
privado; este repositório público de releases não fornece o código-fonte
nem afirma que terceiros possam reproduzir a build. O APK publicado e
seu manifesto permitem verificar manualmente algumas afirmações, como a
identidade da assinatura e as permissões. Os limites da garantia estão
na §12.

## 1. Cifragem

| O quê | Como | Parâmetros |
|-------|------|------------|
| Cifra simétrica | AES-256-GCM (NIST SP 800-38D) | Chave de 32 bytes (256 bits), tag de 16 bytes (128 bits), nonce de 12 bytes (96 bits) por escrita |
| Derivação de chave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iterações, paralelismo 4, sal de 16 bytes, chave derivada de 32 bytes |
| Sal por cofre | Aleatório, 16 bytes | Gerado por uma fonte aleatória criptograficamente segura na criação do cofre |
| Nonce por escrita | Aleatório, 12 bytes | A cada escrita são solicitados 12 bytes novos ao `SecureRandom` do Java |
| Header | 50 bytes | Os primeiros 46 bytes são autenticados como AAD; os 4 bytes finais de `ciphertextLen` não são |

**O que significa "tag".** AES-GCM produz um tag de autenticação de
16 bytes. Ele autentica os primeiros 46 bytes do header como AAD e o
ciphertext declarado. Alterar esse conteúdo faz a verificação falhar.
Os 4 bytes de `ciphertextLen` não são autenticados, e o leitor atual
permite e ignora bytes após o ciphertext declarado; eles não são
cobertos pelo tag.

**O que significa "nonce".** O nonce de 12 bytes é um valor que nunca
deve se repetir sob a mesma chave. Em cada escrita, o app solicita 12
bytes novos ao `SecureRandom` do Java. Não há detector explícito de
repetição nem teste de fonte aleatória defeituosa; é uma proteção
probabilística, não garantia de unicidade absoluta ou verificação
fail-closed.

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

Os primeiros 46 bytes do header são o AAD. A região autenticada é esses
46 bytes mais o ciphertext declarado e seu tag GCM. `ciphertextLen` não
é autenticado. O leitor atual permite e ignora bytes posteriores.

## 3. Substituição na escrita e limites de durabilidade

Cada salvamento usa o padrão escrever-depois-renomear. O app:

1. Escreve o novo cofre em `vault.bvda.tmp`.
2. Sincroniza o descritor do arquivo temporário.
3. Tenta `ATOMIC_MOVE` com substituição.
4. Se falhar, tenta um move não atômico com substituição.
5. Se também falhar, copia sobre o destino e apaga o temporário.

O diretório não é sincronizado. Apenas o primeiro método pretende ser
atômico, conforme o suporte do sistema de arquivos. Os fallbacks não
garantem atomicidade. Uma falha ou perda de energia pode deixar o
destino interrompido ou ausente; não se promete durabilidade total.

## 4. Proteção biométrica da chave

Quando o desbloqueio biométrico está ativado, a chave derivada é
embrulhada com uma chave do Android Keystore. Essa chave não é
exportável pela API Android, exige `BIOMETRIC_STRONG` e é invalidada
quando a biometria é cadastrada novamente.

O suporte por hardware, TEE ou StrongBox depende do dispositivo e do
Keystore. O app não solicita StrongBox nem verifica suporte por hardware;
essas propriedades não devem ser presumidas em todos os dispositivos.

## 5. Sem recuperação de senha

Não há mecanismo de recuperação, nem redefinição por e-mail, nem chave
de recuperação, nem atendimento ao cliente que possa abrir um cofre. A
senha mestra original nunca é armazenada, transmitida ou registrada e
não pode ser recuperada.

Se o desbloqueio biométrico já estava habilitado e continua válido, ele
ainda pode abrir o cofre. Exporte imediatamente um `.bvda` cifrado com
uma nova senha para resgatar seus dados; isso não recupera a senha
mestra original. Se a biometria não estava habilitada, falhar ou for
invalidada antes da exportação, o acesso ao cofre será perdido
definitivamente.

O autor deste projeto também não pode abrir o seu cofre.

## 6. Modelo de ameaças — o que o Brako Vault protege

O app é projetado para proteger contra:

- Roubo do dispositivo desligado, por um adversário sem a senha mestra
  e sem capacidade de contornar a criptografia de disco do Android.
- Ataques pela rede: o app não tem permissão `INTERNET`, então
  condições de rede comprometidas não podem exfiltrar o cofre.
- Modificação do conteúdo BVDA autenticado: mudanças nos primeiros 46
  bytes, no ciphertext declarado ou no tag GCM são detectadas.
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
- Replay ou rollback para um `.bvda` válido anterior. AES-GCM autentica
  conteúdo, mas não sua atualidade ou uma versão crescente.
- Alterações no `ciphertextLen` não autenticado ou em bytes posteriores.
- Exposição da senha de uma exportação ou transferência por canal não
  confiável.

## 7. Backups

O app exporta um `.bvda` no formato cifrado do cofre. Toda exportação
exige uma senha informada pelo usuário, que pode ser a senha mestra ou
outra. Não existe exportação sem senha. Por padrão, o backup do
dispositivo é desativado para manter o cofre fora de `adb backup` e da
nuvem.

## 8. Sincronização offline

O Brako Vault não tem servidor. A sincronização entre dispositivos
funciona trocando arquivos `.bvda` por um canal que o usuário escolhe
(USB, e-mail, armazenamento na nuvem, AirDrop). O app nunca abre um
socket de rede, então o canal não pode ser observado pelo próprio app.
Na importação, AES-GCM verifica o conteúdo BVDA autenticado e detecta
alterações nessa região. Isso não detecta replay de arquivo válido
anterior nem mudanças nos dados não autenticados descritos na §2.

## 9. Permissões declaradas

A partir da v0.4.0, o APK declara exatamente três permissões da
plataforma Android:

| Permissão | Por quê |
|-----------|---------|
| `android.permission.CAMERA` | Para escanear QR codes com tokens 2FA. A câmera é usada apenas quando o usuário abre o scanner de QR; os quadros são processados no dispositivo e nunca são guardados. |
| `android.permission.VIBRATE` | Para dar feedback háptico em ações como desbloqueio bem-sucedido ou copiar para a área de transferência. |
| `android.permission.USE_BIOMETRIC` | Para permitir desbloqueio biométrico opcional, mediado pelo Android Keystore. O app não acessa dados biométricos diretamente. |

O workflow de release verifica com `aapt2` esse conjunto exato de
permissões da plataforma. O AndroidX também adiciona a permissão
personalizada
`com.brakovault.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`. Ela é
privada, com nível de assinatura; não é uma permissão da plataforma
Android nem concede acesso à rede. Na v0.3.0, podiam aparecer permissões
normais adicionais fornecidas pelo AndroidX.

O APK **não declarava e continua sem declarar**
`android.permission.INTERNET`. Não há fallback, não há exceção de
"apenas build debug", nem SDK de terceiros que a solicite. Se uma
feature futura precisar de rede, a feature é reconsiderada; a permissão
não é adicionada.

## 10. Como verificar que o APK não tem permissão INTERNET

Você não precisa confiar neste documento. Você pode verificar em
qualquer dispositivo ou estação de trabalho:

```shell
# Do Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# ou
aapt dump permissions brako-vault-vX.Y.Z.apk
```

Na v0.4.0 e posteriores, as permissões esperadas da plataforma Android
são exatamente as três da §9. O `aapt2` também pode mostrar a permissão
privada do AndroidX descrita ali. Na v0.3.0, podiam aparecer outras
permissões normais fornecidas pelo AndroidX. Se
`android.permission.INTERNET` aparecer em qualquer versão, o APK não
corresponde a este documento; não o instale.

## 11. Procedência e assinatura dos binários

Cada release é assinado com a chave do mantenedor. A partir da v0.4.0,
também inclui `SIGNING-CERTIFICATE.txt` e `SHA256SUMS.txt`; versões
anteriores não incluem esses arquivos. Para inspecionar localmente:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

A impressão digital SHA-256 oficial e canônica do certificado de
assinatura é:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

A partir da v0.4.0, o workflow de release exige essa impressão digital.
A impressão digital SHA-256 exibida pelo `apksigner` deve corresponder
tanto ao valor acima quanto a `SIGNING-CERTIFICATE.txt`, e os hashes
dos artefatos devem corresponder a `SHA256SUMS.txt`.

`SHA256SUMS.txt` e `SIGNING-CERTIFICATE.txt` não têm uma assinatura
separada. Eles permitem detectar corrupção, mas, sozinhos, não detectam
o comprometimento de uma conta ou repositório do GitHub, pois um
atacante poderia substituí-los junto com os artefatos. A assinatura do
APK e a impressão digital fixada acima autenticam o APK; não autenticam
um arquivo AAB nem BLF.

## 12. Níveis de garantia

As evidências disponíveis têm escopos distintos:

- **Declaração de projeto.** Este documento público declara o projeto e
  os parâmetros exatos.
- **Testes internos.** Testes no repositório privado verificam parâmetros
  canônicos em documento privado e comportamento interno. Não provam a
  inspeção destes documentos públicos nem do APK/manifesto publicado.
- **Verificação externa manual.** Qualquer pessoa pode inspecionar
  certificado e permissões com os comandos acima e, desde a v0.4.0,
  comparar hashes e impressão digital com os dois arquivos do release e
  com a impressão digital canônica deste documento, respeitando os
  limites indicados na §11.

O Brako Vault **não** passou por auditoria externa de segurança,
certificação, Common Criteria ou pentest de terceiros. Não há garantia
de build reproduzível.

## 13. Reporte de vulnerabilidades

Para relatos sensíveis, use o
[reporte privado de vulnerabilidades](https://github.com/waar19/brako-vault-releases/security/advisories/new).
Não publique detalhes sensíveis em uma issue. Para dúvidas não
sensíveis, use as [issues públicas](https://github.com/waar19/brako-vault-releases/issues).
Não há promessa de prazo fixo para confirmação ou correção.
