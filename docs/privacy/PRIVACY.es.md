# Extensión de navegador Brako Vault — Política de privacidad

[English](../../PRIVACY.md) | **[Español](PRIVACY.es.md)** | [Português](PRIVACY.pt.md) | [Français](PRIVACY.fr.md) | [Deutsch](PRIVACY.de.md) | [Italiano](PRIVACY.it.md) | [日本語](PRIVACY.ja.md) | [简体中文](PRIVACY.zh-CN.md)

Fecha de vigencia: 9 de septiembre de 2026

## Alcance

Esta política cubre la extensión de navegador Brako Vault para navegadores
basados en Chromium y Firefox, y su host opcional de mensajería nativa para
Windows.

## Datos manejados

Para cumplir su único propósito —usar credenciales de una bóveda local
cifrada— la extensión maneja:

- datos de autenticación, incluidos usuarios y contraseñas;
- el origen y la URL de la página de acceso activa;
- el envío explícito de un formulario al preparar una propuesta de guardado o
  actualización;
- la carpeta de la bóveda y el nombre del dispositivo introducidos en
  Configuración;
- las entradas de la bóveda seleccionadas por el usuario.

La extensión solo lee campos relacionados con el acceso. No recopila el
historial entre pestañas, cookies, datos de pago o salud, identificadores
publicitarios, analytics, reportes de fallos ni telemetría.

## Uso de los datos

Los datos se usan únicamente para desbloquear y buscar en la bóveda local,
mostrar credenciales coincidentes, rellenar los campos elegidos por el usuario
y preparar un guardado o actualización que requiere confirmación. Brako Vault
nunca envía automáticamente un formulario del sitio web.

## Mensajería nativa local

La extensión intercambia los datos anteriores con
`com.brakovault.desktop_host`, un programa de Windows que el usuario instala
por separado. Firefox clasifica Native Messaging como transmisión fuera del
navegador; por eso el manifest de Firefox declara datos de autenticación,
actividad de navegación y actividad en sitios web.

El host es local y no envía datos por la red. Brako Vault, Google, Mozilla y
terceros no reciben la bóveda, las credenciales, las URL, el nombre del
dispositivo, la ruta de la carpeta ni información de uso.

## Almacenamiento y protección

La bóveda permanece en la carpeta elegida por el usuario como el archivo
cifrado `vault.bvda`. El host guarda la configuración en
`%LOCALAPPDATA%\Brako Vault\config.json`. Si se activa el acceso rápido con
Windows Hello, guarda un registro cifrado en
`%LOCALAPPDATA%\Brako Vault\security\quick-unlock.json`; el registro no
contiene en texto claro la contraseña ni la clave maestra.

La bóveda usa AES-256-GCM y Argon2id. Windows Hello verifica al usuario local
de Windows antes del acceso rápido; no sustituye el cifrado de la bóveda ni
garantiza protección respaldada por hardware.

## Compartición, venta y procesamiento remoto

Ningún dato se vende, alquila, comparte con terceros, usa para publicidad,
decisiones de crédito ni procesa en un servicio remoto. El producto no tiene
cuentas de usuario ni sincronización con servidores.

## Conservación y eliminación

Brako Vault no puede acceder ni eliminar datos remotamente. El usuario
controla su conservación:

- desactivar Windows Hello elimina `quick-unlock.json`;
- desinstalar el host elimina sus registros y el acceso rápido, pero conserva
  intencionalmente `config.json` y la bóveda elegida por el usuario;
- el usuario puede eliminar manualmente `config.json` y su bóveda después de
  cerrar el host;
- desinstalar la extensión elimina los datos que administra el navegador para
  ella.

Eliminar la bóveda es irreversible sin otra copia de seguridad cifrada.

## Cambios

Los cambios importantes se publicarán en esta misma URL junto con una
actualización de la extensión. También se actualizará la fecha de vigencia.

## Contacto

Para asuntos de privacidad o seguridad, usa el
[reporte privado de vulnerabilidades de GitHub](https://github.com/waar19/brako-vault-releases/security/advisories/new).
