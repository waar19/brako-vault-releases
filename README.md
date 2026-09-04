# Brako Vault

Repositorio oficial de descargas de Brako Vault para Android.

Brako Vault es un gestor de contraseñas que funciona completamente sin
conexión:

- El APK no declara el permiso `INTERNET`.
- No usa cuentas, telemetría ni sincronización con servidores.
- Protege la bóveda con AES-256-GCM y una clave derivada mediante Argon2id.

## Descargar e instalar

1. Abre la sección [Releases](https://github.com/waar19/brako-vault-releases/releases).
2. Descarga el archivo `.apk` de la versión más reciente.
3. Instálalo en un dispositivo Android con API 29 o superior.
4. Conserva tu contraseña maestra en un lugar seguro. No existe un mecanismo
   de recuperación.

El archivo `.aab` se publica para distribución y validación, pero no se instala
directamente en un dispositivo.

## Verificar la firma

Puedes comprobar la firma del APK con las herramientas oficiales de Android:

```shell
apksigner verify --verbose brako-vault-vX.Y.Z.apk
```

Los datos opcionales para la detección offline de contraseñas filtradas y sus
checksums se publican en
[waar19/brako-vault-data](https://github.com/waar19/brako-vault-data/releases).

## Código fuente

Este repositorio distribuye únicamente binarios oficiales. El código fuente de
Brako Vault no es público ni se incluye en este repositorio.

## English

This repository provides the official Brako Vault Android downloads. The app
works offline, requests no `INTERNET` permission, and includes no accounts,
telemetry, or server synchronization. Download the latest APK from
[Releases](https://github.com/waar19/brako-vault-releases/releases).

This is a binary distribution repository. The Brako Vault source code is not
public and is not included here.
