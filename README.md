# Token Service — GDE Entre Ríos

Adaptación del instalador MSI del componente de firma digital utilizado por GDE, para permitir su utilización desde la extensión de Chrome **Firma con token** en la plataforma GDE Entre Ríos.

## Descripción

Este proyecto contiene el empaquetado MSI del componente nativo utilizado para realizar operaciones de firma digital mediante token/certificado desde Google Chrome.

El instalador original registra un **Native Messaging Host** denominado:

```text
gcbatoken
```

El objetivo de esta adaptación es mantener la implementación nativa existente (`tokensign.exe`) y modificar el manifiesto de Native Messaging para autorizar la extensión de Chrome utilizada por GDE Entre Ríos.

### Extensión de Chrome

```text
ID: pliddliphajaldpdihheppcdmoejfafn
Nombre: Firma con token
```

### Native Messaging Host

```text
Nombre: gcbatoken
Tipo: stdio
Ejecutable: tokensign.exe
```

## Arquitectura

```text
┌──────────────────────────────┐
│       GDE Entre Ríos         │
│      *.entrerios.gov.ar      │
└──────────────┬───────────────┘
               │
               │ External Messaging
               ▼
┌──────────────────────────────┐
│ Chrome Extension             │
│ "Firma con token"            │
│                              │
│ pliddliphajaldpdihheppcdmoejfafn │
└──────────────┬───────────────┘
               │
               │ Native Messaging
               │ connectNative()
               ▼
┌──────────────────────────────┐
│ gcbatoken                    │
│ Native Messaging Host        │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────────────────────┐
│ tokensign.exe                │
│ Componente nativo de firma   │
└──────────────────────────────┘
```

## Archivos principales

```text
.
├── README.md
├── msi/
│   └── token-service-entrerios.msi
├── native-host/
│   └── cnmtoken.json
└── source/
    └── ...
```

> La estructura puede modificarse según las herramientas utilizadas para reconstruir el MSI.

## Modificación realizada

El instalador original contiene un manifiesto `cnmtoken.json` similar a:

```json
{
  "name": "gcbatoken",
  "description": "GCBA Firma Digital",
  "path": "tokensign.exe",
  "type": "stdio",
  "allowed_origins": [
    "chrome-extension://copfinchbmcbjdffipaphabmjnnhijhe/",
    "chrome-extension://dcecmgaholkhgdkebcpgdjemmcmflpmf/",
    "chrome-extension://dmlhldehdinlhohacalhlkpcjigmommf/"
  ]
}
```

La adaptación agrega el ID de la extensión utilizada por GDE Entre Ríos:

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

El nombre del Native Messaging Host **no debe modificarse**, ya que la extensión utiliza:

```javascript
chrome.runtime.connectNative("gcbatoken");
```

## Registro de Windows

Chrome busca el Native Messaging Host mediante:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Google\Chrome\NativeMessagingHosts\gcbatoken
```

El valor predeterminado de esta clave debe apuntar al manifiesto instalado, por ejemplo:

```text
C:\Program Files\GDE Firma Digital\cnmtoken.json
```

El MSI se encarga de crear esta entrada durante la instalación.

## Compatibilidad con la extensión

La extensión utilizada actualmente tiene:

```text
Manifest Version: 3
Extension ID: pliddliphajaldpdihheppcdmoejfafn
Version: 1.3.2
```

Su `main.js` utiliza:

```javascript
var hostName = "gcbatoken";

port = chrome.runtime.connectNative(hostName);
```

Además, antes de enviar una solicitud al componente nativo agrega:

```javascript
request.version = "2.0";
```

Por lo tanto, el nombre del Native Messaging Host y el protocolo utilizado por la extensión deben mantenerse sin cambios.

## Compatibilidad con GDE Entre Ríos

La extensión declara dominios `*.gov.ar` dentro de `externally_connectable`, por lo que los sitios de GDE Entre Ríos bajo dominios `*.entrerios.gov.ar` pueden ser compatibles con la configuración actual de la extensión.

Esto debe verificarse mediante una prueba real de firma.

La modificación del MSI **no modifica automáticamente la extensión de Chrome ni garantiza la compatibilidad funcional del protocolo de firma**.

## Requisitos

* Windows 10/11
* Google Chrome
* Extensión `Firma con token`
* ID de extensión:

```text
pliddliphajaldpdihheppcdmoejfafn
```

* Token/certificado digital compatible
* Permisos administrativos para instalar el MSI

## Instalación

Ejecutar:

```text
token-service-entrerios.msi
```

como administrador.

Una vez instalado, comprobar que exista:

```text
C:\Program Files\GDE Firma Digital\
```

y que contenga:

```text
tokensign.exe
cnmtoken.json
```

También comprobar el registro:

```text
HKEY_LOCAL_MACHINE\SOFTWARE\Google\Chrome\NativeMessagingHosts\gcbatoken
```

El valor debe apuntar a:

```text
cnmtoken.json
```

## Verificación

Después de instalar el MSI:

1. Cerrar todas las ventanas de Chrome.
2. Abrir Chrome nuevamente.
3. Verificar que la extensión **Firma con token** esté instalada.
4. Ingresar a GDE Entre Ríos.
5. Realizar una operación que requiera firma digital.
6. Verificar que Chrome pueda iniciar la comunicación con `gcbatoken`.
7. Comprobar que el token/certificado sea detectado.
8. Realizar una firma de prueba.

## Diagnóstico

Si Chrome informa que no puede conectarse con el Native Messaging Host, comprobar:

### 1. Registro

```cmd
reg query "HKLM\SOFTWARE\Google\Chrome\NativeMessagingHosts\gcbatoken"
```

Debe devolver la ruta al manifiesto.

### 2. Manifiesto

Comprobar:

```text
name = gcbatoken
type = stdio
path = tokensign.exe
```

y que incluya:

```text
chrome-extension://pliddliphajaldpdihheppcdmoejfafn/
```

en `allowed_origins`.

### 3. Ejecutable

Comprobar que exista `tokensign.exe` en la ruta indicada por `path`.

### 4. Arquitectura

Verificar que `tokensign.exe` sea compatible con la arquitectura de Windows utilizada.

## Build del MSI

El MSI original fue construido utilizando:

```text
WiX Toolset 3.9
```

La reconstrucción debería conservar, en lo posible:

* ProductCode
* UpgradeCode
* estructura de componentes
* ejecutable `tokensign.exe`
* Native Messaging Host `gcbatoken`

La modificación principal consiste en actualizar el manifiesto `cnmtoken.json` y generar un nuevo paquete MSI.

## Control de versiones

No reemplazar el MSI original.

Mantener versiones separadas:

```text
original/
    token-service_v4.2.msi

releases/
    token-service-entrerios-1.0.0.msi
```

Esto permite comparar y volver a la versión original en caso de problemas.

## Seguridad

`tokensign.exe` debe considerarse un componente ejecutable de confianza.

Antes de distribuir una nueva versión se recomienda registrar:

```text
SHA-256
```

del MSI y del ejecutable.

Ejemplo:

```powershell
Get-FileHash .\token-service-entrerios-1.0.0.msi -Algorithm SHA256
```

También se recomienda comprobar la firma digital del ejecutable y del instalador antes de distribuirlos.

## Estado del proyecto

### Investigado

* [x] Identificación del MSI
* [x] Identificación del Native Messaging Host
* [x] Identificación de `tokensign.exe`
* [x] Identificación de `cnmtoken.json`
* [x] Identificación de la extensión Chrome
* [x] Comparación del nombre `gcbatoken`
* [x] Identificación de `allowed_origins`
* [x] Identificación del ID de extensión de Entre Ríos

### Pendiente

* [ ] Extraer y analizar `tokensign.exe`
* [ ] Confirmar el protocolo de comunicación
* [ ] Modificar `cnmtoken.json`
* [ ] Generar MSI
* [ ] Instalar en entorno de prueba
* [ ] Probar comunicación Chrome → Native Host
* [ ] Probar detección del token
* [ ] Probar firma en GDE Entre Ríos
* [ ] Validar instalación/desinstalación
* [ ] Generar release

## Licencia y procedencia

Este repositorio contiene una adaptación de un instalador existente. Antes de distribuir públicamente el MSI modificado, verificar que se cuente con autorización para redistribuir `tokensign.exe` y los demás componentes originales.

---

**Proyecto:** Token Service — GDE Entre Ríos
**Native Host:** `gcbatoken`
**Chrome Extension:** `pliddliphajaldpdihheppcdmoejfafn`
