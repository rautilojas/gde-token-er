# Token Service — GDE Entre Ríos

Adaptación del instalador MSI del componente de firma digital nativo utilizado por el sistema GDE. Esta versión modifica los orígenes permitidos para habilitar la comunicación desde la extensión de Chrome **Firma con token** específica de la plataforma GDE Entre Ríos.

## Descripción

Este proyecto contiene el proceso de reempaquetado del instalador MSI original. El componente instala y registra un **Native Messaging Host** en el sistema operativo, el cual sirve de puente entre el navegador y el dispositivo criptográfico (token USB).

### Extensión de Chrome (Entre Ríos)
* **ID:** `pliddliphajaldpdihheppcdmoejfafn`
* **Nombre:** Firma con token

### Native Messaging Host
* **Nombre:** `gcbatoken`
* **Ejecutable local:** `tokensign.exe`
* **Tipo:** `stdio`

---

## Arquitectura de Comunicación

El flujo de firma digital requiere que la página web se comunique con la extensión, y que esta, a su vez, invoque el binario local instalado por este MSI.

```text
┌──────────────────────────────┐
│       GDE Entre Ríos         │
│     *.entrerios.gov.ar       │
└──────────────┬───────────────┘
               │
               │ (externally_connectable)
               ▼
┌──────────────────────────────┐
│ Chrome Extension             │
│ "Firma con token"            │
│ ID: pliddliphajaldpdih...    │
└──────────────┬───────────────┘
               │
               │ chrome.runtime.connectNative("gcbatoken")
               ▼
┌──────────────────────────────┐
│ gcbatoken                    │
│ (Native Messaging Host)      │
└──────────────┬───────────────┘
               │
               │ (Validación vía cnmtoken.json)
               ▼
┌──────────────────────────────┐
│ tokensign.exe                │
│ (Interacción con Token USB)  │
└──────────────────────────────┘

```

---

## Modificación Realizada

El instalador original restringe la comunicación a un conjunto cerrado de extensiones mediante el archivo de manifiesto `cnmtoken.json`.

La adaptación desempaqueta el MSI original y agrega el ID de la extensión de GDE Entre Ríos al array `allowed_origins`, manteniendo intacto el binario ejecutable y el resto del comportamiento.

**Manifiesto modificado (`cnmtoken.json`):**

```json
{
  "name": "gcbatoken",
  "description": "GDE Firma Digital",
  "path": "tokensign.exe",
  "type": "stdio",
  "allowed_origins": [
    "chrome-extension://copfinchbmcbjdffipaphabmjnnhijhe/",
    "chrome-extension://dcecmgaholkhgdkebcpgdjemmcmflpmf/",
    "chrome-extension://dmlhldehdinlhohacalhlkpcjigmommf/",
    "chrome-extension://pliddliphajaldpdihheppcdmoejfafn/"
  ]
}

```

*Nota: El nombre interno del host (`gcbatoken`) no fue alterado, ya que el código de la extensión busca explícitamente esa nomenclatura para establecer la conexión.*

---

## Instalación y Verificación

1. Descargar y ejecutar `token-service-er.msi` con permisos de Administrador.
2. Comprobar que los archivos `tokensign.exe` y `cnmtoken.json` se hayan extraído en el directorio de instalación correspondiente en `Program Files`.
3. Verificar la creación de la clave de registro que enlaza Chrome con el host local:
```cmd
reg query "HKLM\SOFTWARE\Google\Chrome\NativeMessagingHosts\gcbatoken"

```


*(El valor devuelto debe ser la ruta absoluta hacia el archivo `cnmtoken.json`)*.

---

## Diagnóstico de Errores

Si Chrome informa que no puede establecer comunicación con el Native Messaging Host o la firma falla:

1. **Validar extensión:** Confirmar que el ID de la extensión instalada en el navegador coincida exactamente con `pliddliphajaldpdihheppcdmoejfafn`.
2. **Validar manifiesto:** Asegurarse de que el archivo `cnmtoken.json` instalado no contenga errores de sintaxis y tenga la ruta correcta hacia `tokensign.exe`.
3. **Drivers del Token:** Comprobar que los controladores del dispositivo criptográfico (SafeNet, ePass, etc.) estén instalados y detectando el hardware en el sistema operativo local.

---

## Build del MSI modificado

El paquete original fue construido con **WiX Toolset**. Para generar nuevas versiones sin romper dependencias internas:

1. Extraer los binarios y el código fuente `.wxs` usando `dark.exe`.
2. Modificar los archivos necesarios (ej. `cnmtoken.json`).
3. Recompilar los objetos con `candle.exe`.
4. Generar el instalador final con `light.exe`.

Se conservaron el `ProductCode` y `UpgradeCode` originales para garantizar que el sistema lo reconozca como una actualización o reemplazo válido de la versión anterior.

---

## Estado del Proyecto

### Completado ✔️

* [x] Identificación del flujo nativo (`gcbatoken` -> `tokensign.exe`).
* [x] Análisis del manifiesto de la extensión de Entre Ríos (`externally_connectable`).
* [x] Descompilación del paquete MSI original.
* [x] Modificación del manifiesto de Native Messaging (`allowed_origins`).
* [x] Recompilación y empaquetado del nuevo `token-service-er.msi`.
* [x] Despliegue en repositorio para descarga directa.
* [x] Configuración y validación de redirección de descarga en entorno GDE de prueba.

### Pendiente ⏳

* [ ] Ejecutar pruebas de carga del certificado con token físico (SafeNet/Token USB).
* [ ] Validar operación de firma digital completa dentro del módulo GEDO por parte de los funcionarios.

---

## Aviso de Seguridad y Procedencia

Este repositorio distribuye una adaptación técnica orientada a mantener compatibilidad en entornos GDE. El ejecutable `tokensign.exe` es el binario original proveído en las distribuciones oficiales. Debido a que el reempaquetado invalida la firma digital original del instalador, los sistemas de seguridad como Windows SmartScreen pueden emitir una advertencia durante el primer despliegue.

```

```
