# Brako Vault — Seguridad y transparencia

[English](../SECURITY.md) | **[Español](SECURITY.es.md)** | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)**

Última actualización: 2026-09-05

Este documento describe cómo Brako Vault protege los datos en tu
dispositivo. Cada afirmación factual de aquí se deriva del
[archivo canónico de parámetros criptográficos](https://github.com/waar19/brako-vault/blob/main/docs/canonical-crypto.md)
público del repositorio de código, o se puede verificar directamente
sobre el APK publicado y el manifest de la aplicación. Donde una
afirmación es "diseñada" pero no "auditada", el documento lo dice.

> **El código fuente es privado.** El código de la aplicación no está
> en este repositorio. Los números, tamaños y protocolos de abajo son
> la declaración pública autoritativa de lo que hace la app. El APK
> publicado se compila a partir del mismo código que contiene las
> constantes en `Argon2Spec` y `BvdaConstants` del repositorio
> privado.

## 1. Cifrado

| Qué | Cómo | Parámetros |
|-----|------|------------|
| Cifrado simétrico | AES-256-GCM (NIST SP 800-38D) | Clave de 256 bits, tag de autenticación de 128 bits, nonce de 96 bits (12 bytes) por escritura |
| Derivación de clave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iteraciones, 4 lanes, sal de 16 bytes, clave derivada de 32 bytes |
| Sal por bóveda | Aleatoria, 16 bytes | Generada por una fuente aleatoria criptográficamente segura al crear la bóveda |
| Nonce por escritura | Aleatorio, 12 bytes | Único por escritura, nunca reutilizado con la misma clave |
| Integridad del header | Ligada al ciphertext | 46 bytes del header autenticados como Additional Authenticated Data (AAD) |

**Qué significa "tag".** AES-GCM produce un tag de autenticación de
128 bits que se añade al ciphertext. Cualquier modificación del archivo
(magic, parámetros KDF, sal, nonce o ciphertext) hace que la
verificación del tag falle y la bóveda rechaza abrirse.

**Qué significa "nonce".** El nonce de 12 bytes es un valor que no debe
repetirse jamás con la misma clave. La app genera un nonce aleatorio
nuevo en cada guardado. Reutilizar un nonce sería catastrófico; por
eso la implementación falla explícitamente de forma segura si la fuente
aleatoria está rota.

## 2. Formato de archivo

Todos los datos de la bóveda se almacenan en un único archivo:
`vault.bvda`.

Disposición (big-endian para campos multi-byte, excepto `saltLen` y
`nonceLen` que son 1 byte):

```
 offset  tamaño  campo
 ------  ------  -----
   0       4    magic            = "BVDA" (ASCII)
   4       1    formatVersion    = 0x01
   5       1    kdfId            = 0x01 (Argon2id)
   6       1    aeadId           = 0x01 (AES-256-GCM)
   7       1    kdfParallelism   = 0x04
   8       4    kdfMemoryKiB     BE = 65 536
  12       4    kdfIterations    BE = 3
  16       1    saltLen          = 0x10 (16)
  17      16    kdfSalt
  33       1    nonceLen         = 0x0C (12)
  34      12    aeadNonce
  46       4    ciphertextLen    BE  <- NO forma parte del AAD
  50       N    ciphertext (N = ciphertextLen, incluye tag de 16 B)
```

Los primeros 46 bytes del header son el AAD. El tag es parte del
ciphertext, por lo que el contenido autenticado del archivo cubre todo
excepto el campo de 4 bytes con la longitud del ciphertext.

## 3. Escritura atómica

Cada guardado sigue el patrón escribir-luego-renombrar. La app:

1. Escribe la nueva bóveda en `vault.bvda.tmp`.
2. Llama a `fsync` sobre el archivo temporal.
3. Llama a `fsync` sobre el directorio.
4. Renombra atómicamente `vault.bvda.tmp` a `vault.bvda`.

Si el dispositivo pierde energía o la app se mata entre pasos, el
`vault.bvda` anterior queda intacto y el archivo temporal queda como
basura (se limpia en el siguiente guardado exitoso).

## 4. Protección biométrica de la clave

Cuando se activa el desbloqueo biométrico, la clave derivada se envuelve
con una clave guardada en Android Keystore. La clave custodiada por
Keystore:

- Nunca sale del hardware seguro cuando el dispositivo tiene un
  Trusted Execution Environment (TEE) o un StrongBox Keymaster.
- No es extraíble por procesos en modo usuario.
- Se invalida cuando el usuario elimina todas las biometrias del
  dispositivo, cambia la pantalla de bloqueo, o se hace factory-reset.

El prompt biométrico lo impone el sistema operativo; la app no puede
saltarlo.

## 5. Sin recuperación de contraseña

No hay mecanismo de recuperación, ni reset por email, ni clave de
recuperación, ni servicio de atención al cliente que pueda abrir una
bóveda. La contraseña maestra es la única clave, nunca se almacena,
nunca se transmite, nunca se registra. Si pierdes la contraseña
maestra, la bóveda desaparece.

El autor de este proyecto tampoco puede abrir tu bóveda.

## 6. Modelo de amenazas — qué protege Brako Vault

La app está diseñada para proteger contra:

- Robo del dispositivo apagado, por un adversario sin la contraseña
  maestra y sin capacidad para saltarse el cifrado de disco de Android.
- Ataques por red: la app no tiene permiso `INTERNET`, por lo que
  condiciones de red comprometidas no pueden exfiltrar la bóveda.
- Replay o modificación del archivo de bóveda: cualquier cambio de un
  bit se detecta por la verificación del tag AES-GCM.
- Fuerza bruta sobre la contraseña maestra: Argon2id con 64 MiB y 3
  iteraciones hace que cada intento sea caro; un atacante offline
  todavía tiene que adivinar la contraseña.

La app **no** está diseñada para proteger contra:

- Un dispositivo comprometido o rooteado que se ejecuta mientras la
  bóveda está abierta. Una vez verificada la contraseña maestra, la
  clave está en memoria y el proceso tiene acceso al plaintext.
- Mirones, keyloggers o malware que se ejecute con el mismo UID que
  la app.
- Coerción: un atacante decidido con acceso físico a un dispositivo
  desbloqueado puede leer la bóveda.
- Una contraseña maestra débil o reutilizada. Argon2id hace más lento
  el ataque pero no puede hacer seguro "123456".
- El usuario que exporta voluntariamente la bóveda sin cifrar a un
  tercero.

## 7. Backups

La app exporta un archivo `.bvda` (el mismo formato cifrado que la
bóveda en disco). El archivo exportado se cifra con la misma clave
derivada de la contraseña maestra, a menos que el usuario elija
explícitamente "exportar sin contraseña" — y en ese caso la app
advierte en voz alta de que el archivo queda desprotegido. Por
defecto, el sistema de backup del dispositivo está desactivado para
mantener la bóveda fuera de los archivos de `adb backup` y de los
backups en la nube.

## 8. Sincronización offline

Brako Vault no tiene servidor. La sincronización entre dispositivos
funciona intercambiando archivos `.bvda` por un canal que el usuario
elige (USB, email, almacenamiento en la nube, AirDrop). La app nunca
abre un socket de red, por lo que el canal no puede ser observado por
la propia app. El archivo se autentica de extremo a extremo por la
misma verificación del tag AES-GCM, así que un tercero que retransmite
una copia alterada es detectado al importar.

## 9. Permisos declarados

El APK declara exactamente tres permisos:

| Permiso | Por qué |
|---------|---------|
| `android.permission.CAMERA` | Para escanear códigos QR con tokens 2FA. La cámara se usa solo cuando el usuario abre el escáner QR; los fotogramas se procesan en el dispositivo y nunca se guardan. |
| `android.permission.VIBRATE` | Para dar respuesta háptica en acciones como desbloqueo exitoso o copiar al portapapeles. |
| `android.permission.USE_BIOMETRIC` | Para permitir el desbloqueo biométrico opcional, mediado por Android Keystore. La app no accede a datos biométricos directamente. |

El APK **no** declara `android.permission.INTERNET`. No hay fallback,
no hay excepción de "solo build debug", ni SDK de terceros que lo
pida. Si una feature futura necesitara red, se reconsidera la feature;
el permiso no se añade.

## 10. Cómo verificar que el APK no tiene permiso INTERNET

No necesitas confiar en este documento. Puedes verificarlo en
cualquier dispositivo o equipo:

```shell
# Desde Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# o
aapt dump permissions brako-vault-vX.Y.Z.apk
```

La salida debe listar únicamente los tres permisos de §9. Si aparece
`android.permission.INTERNET`, el archivo no es el APK oficial — no
lo instales.

## 11. Procedencia y firma de los binarios

Cada release está firmado con la clave de release del mantenedor. La
huella se publica en las notas del release. Para verificar en local:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

La huella SHA-256 del certificado de firma debe coincidir con la de
las notas del release. Los checksums de cada artefacto están en
`SHA256SUMS.txt`, junto a los binarios del mismo release.

## 12. Niveles de garantía

Brako Vault se ofrece con tres niveles de garantía explícitos. No
son lo mismo:

- **Diseñado.** La arquitectura y los parámetros de este documento son
  lo que el autor se comprometió a implementar. Es la afirmación más
  débil.
- **Probado automáticamente.** Una batería de tests en el repositorio
  privado de código — `CryptoSpecDocTest` y el resto de `jvmTest` y
  `androidApp:testDebugUnitTest` — corre en cada push y verifica que la
  implementación coincide con lo declarado aquí, incluidos los
  parámetros criptográficos y la ausencia de `INTERNET` en el
  manifest.
- **Auditado externamente.** Brako Vault **no** ha sido auditado por
  una parte externa. El autor no afirma certificación externa,
  evaluación de common criteria ni pentest de terceros. Si se hiciera
  una auditoría futura, sus hallazgos se publicarán aquí con fecha,
  alcance e informe completo.

## 13. Reporte de vulnerabilidades

Si encuentras una vulnerabilidad, escribe a
`security@brakovault.example` (sustituye por la dirección real cuando
el proyecto la haga pública). No abras un issue público de GitHub
para reportes sensibles de seguridad. El autor se compromete a
confirmar recepción en 72 horas y a proporcionar un arreglo o una
aceptación documentada del riesgo en 30 días para los problemas
confirmados.
