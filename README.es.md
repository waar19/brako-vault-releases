# Brako Vault

[English](README.md) | **[Español](README.es.md)** | [Português](README.pt.md) | [Français](README.fr.md) | [Deutsch](README.de.md) | [Italiano](README.it.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md)

Repositorio oficial de descargas de Brako Vault para Android.

Brako Vault es un gestor de contraseñas que funciona completamente sin
conexión:

- El APK no declara el permiso `INTERNET`.
- No usa cuentas, telemetría ni sincronización con servidores.
- Protege la bóveda con AES-256-GCM y una clave derivada mediante Argon2id.
- La contraseña maestra no se puede recuperar. Si la pierdes, la bóveda
  desaparece.

## Descargar e instalar

1. Abre la sección [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Descarga el archivo `.apk` de la versión más reciente.
3. Instálalo en un dispositivo Android con API 29 (Android 10) o superior.
4. Conserva tu contraseña maestra en un lugar seguro. No existe un
   mecanismo de recuperación.

> **Si vienes de una build de desarrollo** (Android Studio ejecutando
> `app` directamente, o un APK firmado con la llave debug de Android
> Studio), desinstálala **antes** de instalar la build firmada de
> release. Las firmas son distintas y Android bloqueará la actualización
> con `INSTALL_FAILED_UPDATE_INCOMPATIBLE` (los gestores como Obtainium
> muestran el error como "FailureConflict"). Pasos:
>
> 1. Ajustes → Aplicaciones → Brako Vault → Desinstalar.
> 2. Instalar la build de release desde aquí.
> 3. Si tenías bóveda, reimportar el `.bvda` (no se copia entre firmas).

## APK vs AAB

- **APK** (Android Package): el archivo que se instala en un dispositivo.
- **AAB** (Android App Bundle): el formato que espera Google Play;
  contiene el mismo código dividido por configuración del dispositivo.
  No se puede hacer side-load de un `.aab`, por eso este repositorio
  publica un `.apk` además del `.aab`.

## Verificar la firma

Puedes comprobar la firma del APK con las herramientas oficiales de
Android:

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

## Verificar el checksum SHA-256

Cada release publica un archivo `SHA256SUMS.txt` con los resúmenes de
cada artefacto. Para verificarlo en Windows (PowerShell):

```powershell
Get-FileHash .\brako-vault-vX.Y.Z.apk -Algorithm SHA256
```

En macOS / Linux:

```shell
sha256sum -c SHA256SUMS.txt
```

Los resúmenes publicados deben coincidir byte a byte con los del
`SHA256SUMS.txt`. Si no coinciden, no instales el archivo.

## Sobre los archivos "Source code" auto-generados

GitHub produce automáticamente los enlaces `Source code (zip)` y
`Source code (tar.gz)` en cada release. Esos archivos se generan a partir
de **este** repositorio y contienen únicamente el README, el aviso de
seguridad y la configuración del repositorio. **No** contienen el
código fuente de la aplicación, que es privado. Tóralos como
documentación, no como código.

## Filtro de contraseñas filtradas (opcional)

La base offline opcional de contraseñas filtradas y sus checksums se
publican en
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Código fuente

Este repositorio distribuye únicamente binarios oficiales. El código
fuente de Brako Vault no es público ni se incluye en este repositorio.
