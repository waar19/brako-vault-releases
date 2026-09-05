# Brako Vault — Seguridad y transparencia

[English](../../SECURITY.md) | **[Español](SECURITY.es.md)** | [Português](SECURITY.pt.md) | [Français](SECURITY.fr.md) | [Deutsch](SECURITY.de.md) | [Italiano](SECURITY.it.md) | [日本語](SECURITY.ja.md) | [简体中文](SECURITY.zh-CN.md)

Última actualización: 2026-09-05

Este documento autocontenido es la declaración pública de seguridad y
transparencia de Brako Vault. El repositorio del código de la aplicación
es privado; este repositorio público de releases no ofrece el código
fuente ni afirma que terceros puedan reproducir la build. El APK
publicado y su manifest permiten comprobar manualmente algunas
afirmaciones, como la identidad de firma y los permisos. Los límites de
garantía se indican en §12.

## 1. Cifrado

| Qué | Cómo | Parámetros |
|-----|------|------------|
| Cifrado simétrico | AES-256-GCM (NIST SP 800-38D) | Clave de 32 bytes (256 bits), tag de autenticación de 16 bytes (128 bits), nonce de 12 bytes (96 bits) por escritura |
| Derivación de clave | Argon2id (RFC 9106) | 64 MiB (65 536 KiB), 3 iteraciones, paralelismo 4, sal de 16 bytes, clave derivada de 32 bytes |
| Sal por bóveda | Aleatoria, 16 bytes | Generada por una fuente aleatoria criptográficamente segura al crear la bóveda |
| Nonce por escritura | Aleatorio, 12 bytes | En cada escritura se solicitan 12 bytes nuevos a `SecureRandom` de Java |
| Header | 50 bytes | Los primeros 46 bytes se autentican como Additional Authenticated Data (AAD); los 4 bytes finales de `ciphertextLen` no |

**Qué significa "tag".** AES-GCM produce un tag de autenticación de
16 bytes. El tag autentica los primeros 46 bytes del header como AAD y
el ciphertext declarado. Una modificación de ese contenido autenticado
hace que la verificación falle. Los 4 bytes de `ciphertextLen` no están
autenticados y el lector actual permite e ignora bytes posteriores al
ciphertext declarado; esos bytes no están cubiertos por el tag.

**Qué significa "nonce".** El nonce de 12 bytes es un valor que no debe
repetirse con la misma clave. En cada escritura, la app solicita 12 bytes
nuevos a `SecureRandom` de Java. La implementación no tiene un detector
explícito de repeticiones ni comprueba si la fuente aleatoria es
defectuosa, por lo que es una protección probabilística, no una garantía
de unicidad absoluta ni una comprobación que falle de forma segura.

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

Los primeros 46 bytes del header son el AAD. El tag forma parte del
ciphertext. La región autenticada son, por tanto, los primeros 46 bytes
más el ciphertext declarado y su tag GCM. `ciphertextLen` no está
autenticado. El lector actual permite e ignora bytes posteriores al
ciphertext declarado.

## 3. Sustitución al escribir y límites de durabilidad

Cada guardado sigue el patrón escribir-luego-renombrar. La app:

1. Escribe la nueva bóveda en `vault.bvda.tmp`.
2. Sincroniza el descriptor del archivo temporal.
3. Intenta `ATOMIC_MOVE` con reemplazo.
4. Si falla, intenta un movimiento no atómico con reemplazo.
5. Si también falla, copia el archivo temporal sobre el destino y
   elimina el temporal.

El directorio no se sincroniza. Solo el primer método pretende ser
atómico, y que funcione depende del sistema de archivos. Los fallbacks de
movimiento y copia/eliminación no garantizan la sustitución atómica. Por
ello, un cierre o corte eléctrico aún puede dejar un destino interrumpido
o ausente cuando se usa un fallback; no se promete durabilidad total
ante fallos.

## 4. Protección biométrica de la clave

Cuando se activa el desbloqueo biométrico, la clave derivada se envuelve
con una clave guardada en Android Keystore. La clave de Keystore no es
exportable mediante la API de Android, exige `BIOMETRIC_STRONG` y está
configurada para invalidarse al volver a registrar biometría.

El respaldo por hardware, TEE o StrongBox depende del dispositivo y de
su implementación de Keystore. La app no solicita StrongBox ni comprueba
que la clave tenga respaldo por hardware. Por tanto, no se deben suponer
esas propiedades en todos los dispositivos compatibles.

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
- Modificación del contenido BVDA autenticado: se detectan cambios en los
  primeros 46 bytes del header, el ciphertext declarado o el tag GCM.
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
- Replay o rollback a un `.bvda` anterior válido. AES-GCM autentica el
  contenido, pero no demuestra frescura ni una versión creciente.
- Cambios en `ciphertextLen` no autenticado o en bytes posteriores al
  ciphertext declarado. Pueden afectar al parseo, pero el tag GCM no los
  cubre.
- Revelar la contraseña de un archivo exportado o transferirlo por un
  canal no fiable.

## 7. Backups

La app exporta un archivo `.bvda` en el formato cifrado de la bóveda.
Cada exportación exige que el usuario introduzca una contraseña para ese
archivo. Puede elegir la contraseña maestra u otra distinta; no existe
exportación sin contraseña. Por defecto, el sistema de backup del
dispositivo está desactivado para mantener la bóveda fuera de los
archivos de `adb backup` y de los backups en la nube.

## 8. Sincronización offline

Brako Vault no tiene servidor. La sincronización entre dispositivos
funciona intercambiando archivos `.bvda` por un canal que el usuario
elige (USB, email, almacenamiento en la nube, AirDrop). La app nunca
abre un socket de red, por lo que el canal no puede ser observado por
la propia app. Al importar se comprueba con AES-GCM el contenido BVDA
autenticado, por lo que se detecta la manipulación dentro de esa región.
Esto no detecta el replay de un archivo anterior válido ni cambios en
los datos de longitud o posteriores no autenticados descritos en §2.

## 9. Permisos declarados

Desde v0.4.0, el APK declara exactamente tres permisos de plataforma
Android:

| Permiso | Por qué |
|---------|---------|
| `android.permission.CAMERA` | Para escanear códigos QR con tokens 2FA. La cámara se usa solo cuando el usuario abre el escáner QR; los fotogramas se procesan en el dispositivo y nunca se guardan. |
| `android.permission.VIBRATE` | Para dar respuesta háptica en acciones como desbloqueo exitoso o copiar al portapapeles. |
| `android.permission.USE_BIOMETRIC` | Para permitir el desbloqueo biométrico opcional, mediado por Android Keystore. La app no accede a datos biométricos directamente. |

El workflow de release verifica con `aapt2` este conjunto exacto de
permisos de plataforma. AndroidX también añade el permiso personalizado
`com.brakovault.app.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION`. Es un
permiso privado, de nivel de firma; no es un permiso de plataforma
Android ni concede acceso a la red. En v0.3.0 podían aparecer permisos
normales adicionales aportados por AndroidX.

El APK **no tenía ni tiene** declarado `android.permission.INTERNET`.
No hay fallback, no hay excepción de "solo build debug", ni SDK de
terceros que lo pida. Si una feature futura necesitara red, se
reconsidera la feature; el permiso no se añade.

## 10. Cómo verificar que el APK no tiene permiso INTERNET

No necesitas confiar en este documento. Puedes verificarlo en
cualquier dispositivo o equipo:

```shell
# Desde Android SDK build-tools:
aapt2 dump permissions brako-vault-vX.Y.Z.apk
# o
aapt dump permissions brako-vault-vX.Y.Z.apk
```

En v0.4.0 y versiones posteriores, los permisos de plataforma Android
esperados son exactamente los tres de §9. `aapt2` también puede mostrar
el permiso privado de AndroidX descrito allí. En v0.3.0 podían aparecer
otros permisos normales aportados por AndroidX. Si
`android.permission.INTERNET` aparece en cualquier versión, el APK no
coincide con este documento; no lo instales.

## 11. Procedencia y firma de los binarios

Cada release está firmado con la clave de release del mantenedor. A
partir de v0.4.0, el release también incluye
`SIGNING-CERTIFICATE.txt` y `SHA256SUMS.txt`. Las versiones anteriores no
incluyen esos dos archivos. Para inspeccionar el APK en local:

```shell
apksigner verify --verbose --print-certs brako-vault-vX.Y.Z.apk
```

La huella SHA-256 oficial y canónica del certificado de firma es:

`8a725a09dbe2483e2cd39435dc53f0355900744c01c97a2e0a860d6118908fbb`

Desde v0.4.0, el workflow de release exige esta huella. La huella
SHA-256 mostrada por `apksigner` debe coincidir tanto con el valor
anterior como con `SIGNING-CERTIFICATE.txt`. Los hashes de los
artefactos deben coincidir con `SHA256SUMS.txt`.

`SHA256SUMS.txt` y `SIGNING-CERTIFICATE.txt` no tienen una firma
separada. Permiten detectar corrupción, pero por sí solos no detectan
el compromiso de una cuenta o repositorio de GitHub, porque un atacante
podría sustituirlos junto con los artefactos. La firma del APK y la
huella anclada arriba autentican el APK; no autentican archivos AAB ni
BLF.

## 12. Niveles de garantía

Las evidencias disponibles tienen alcances distintos:

- **Declaración de diseño.** Este documento público declara el diseño de
  seguridad previsto y los parámetros exactos.
- **Pruebas internas.** Los tests del repositorio privado comprueban los
  parámetros canónicos en un documento privado y el comportamiento
  interno. No demuestran que se hayan inspeccionado estos documentos
  públicos ni que se hayan probado el APK publicado o su manifest.
- **Verificación externa manual.** Cualquiera puede inspeccionar
  manualmente el certificado y los permisos declarados de un APK
  descargado con los comandos anteriores y, desde v0.4.0, comparar sus
  hashes y huella con los dos archivos del release y con la huella
  canónica de este documento, con los límites indicados en §11.

Brako Vault **no** ha recibido una auditoría de seguridad externa,
certificación, evaluación Common Criteria ni pentest de terceros. No se
afirma que la build sea reproducible.

## 13. Reporte de vulnerabilidades

Para reportes sensibles, usa el reporte privado de vulnerabilidades de
GitHub: [reportar una vulnerabilidad de forma privada](https://github.com/waar19/brako-vault-releases/security/advisories/new).
No incluyas detalles sensibles de seguridad en un issue público. Para
dudas no sensibles, puedes usar los
[issues](https://github.com/waar19/brako-vault-releases/issues)
públicos del repositorio. No se promete un plazo fijo de acuse de recibo
ni de corrección.
