# 🚀 Facturama API Collections — Postman & Bruno

> [!NOTE]
> This document is also available in [English](README-en.md).

Colecciones de la API de [Facturama](https://api.facturama.mx/) para probar, entender e
integrar los endpoints de facturación electrónica (CFDI 4.0) sin escribir una sola línea
de código.

El mismo conjunto de requests se publica en **dos formatos**:

| Cliente | Carpeta | Formato |
|---|---|---|
| **Postman** | [postman/](postman/) | Collection v2.1 (`FacturamaAPI.postman_collection_v2.1.json`) + environments |
| **Bruno** | [bruno/](bruno/) | OpenCollection 1.0 (carpetas + `.yml`, versionable en Git) |

Guía completa de la API: <https://apisandbox.facturama.mx/guias>

---

## ⚡ Quick Start

### Postman

1. Clona o descarga este repositorio.
2. **Import** → `postman/FacturamaAPI.postman_collection_v2.1.json`.
3. **Import** → los dos entornos de `postman/Environments/`:
   - `Facturama API Test.postman_environment.json`
   - `Facturama API Prod.postman_environment.json`
4. Selecciona el entorno **Facturama API Test** y llena `UserName` y `Password`.
5. Ejecuta cualquier request.

### Bruno

1. Clona o descarga este repositorio.
2. **Open Collection** → selecciona la carpeta `bruno/` (la que contiene `opencollection.yml`).
   No se importa un archivo: Bruno abre la carpeta tal cual y los cambios quedan en Git.
3. Selecciona el entorno **Sandbox** y llena `Password` y el `UserName`.
4. Ejecuta cualquier request.

> Bruno debe soportar el formato **OpenCollection 1.0**. Si al abrir la carpeta no
> aparecen las requests, actualiza Bruno a la última versión.

✅ En pocos minutos puedes hacer operaciones reales contra la API.

---

## 🔐 Autenticación

Todas las requests usan **HTTP Basic Auth** con las credenciales de tu cuenta Facturama,
más un header `User-Agent` con tu usuario:

| Variable | Uso |
|---|---|
| `SERVER` | Host base de la API (según el entorno) |
| `UserName` | Usuario de la cuenta → Basic Auth + header `User-Agent` |
| `Password` | Contraseña de la cuenta → Basic Auth |

En Postman la auth está definida **por request** (no heredada de la colección); en Bruno
las carpetas usan `auth: inherit`. En ambos casos se deshabilita el `User-Agent`
automático del cliente para enviar el propio.

> [!WARNING]
> Nunca hagas commit de credenciales reales. En Bruno, `Password` ya está marcado como
> `secret` y no se serializa al archivo del entorno. En Postman, marca el valor como
> **secret** en el environment antes de guardarlo o exportarlo.

---

## 🌍 Entornos

| Ambiente | Postman | Bruno | `SERVER` |
|---|---|---|---|
| Pruebas | `Facturama API Test` | `Sandbox` | `https://apisandbox.facturama.mx` |
| Producción | `Facturama API Prod` | `Producción` | `https://api.facturama.mx` |

Los nombres difieren entre clientes, pero las variables y los hosts son los mismos.
Selecciona el entorno **antes** de ejecutar cualquier request.

### Cuenta de pruebas

- Crea una cuenta en el ambiente [Sandbox](https://dev.facturama.mx/api/registro).
- Para **API Web**, configura el perfil fiscal con el RFC de pruebas **`EKU9003173C9`**
  ([guía](https://apisandbox.facturama.mx/guias/perfil-fiscal)).
- Sellos digitales de prueba (CSD): [facturama-CSD-prueba](https://apisandbox.facturama.mx/guias/conocimientos/sellos-digitales-pruebas).

---

## 🧩 Estructura

Las colecciones se dividen en las dos familias de la API de Facturama:

### `API Web` — emisor único

Opera sobre la cuenta propia (el perfil fiscal configurado) y **todo se refleja en la
plataforma web** de Facturama.

| Módulo | Contenido |
|---|---|
| `Clientes` | Alta, edición, borrado, listados, filtros por RFC, validación y status de RFC |
| `Productos ó servicios` | Alta/edición/borrado, listados paginados, productos con IVA, IEPS, ISR, retenciones, cuenta predial |
| `CFDI 4.0` | Ingreso, arrendamiento, notas de crédito, factura global, público en general, moneda extranjera, IEPS, IVA exento/tasa 0, estímulo fronterizo, cliente extranjero, consulta y filtrado, status SAT, descarga PDF/XML/acuse, envío por email, cancelación |
| `Complementos` | Pagos (múltiples variantes), Carta Porte 3.1 (autotransporte, aéreo, marítimo, ferroviario, internacional), Comercio Exterior, Nómina, INE, IEDU, Donativos, Hidrocarburos, Leyendas Fiscales, Notarios Públicos |
| `Retenciones` | Emisión v2 y plataformas tecnológicas, detalle, filtros, descarga y cancelación |
| `Mi cuenta` | Datos de facturación, `TaxEntityInfo`, CSD/FIEL, logo, series, lugares de expedición, suscripción |
| `Catalogos` | Catálogos del SAT: generales, Carta Porte y Nómina |

### `API-Lite` — multiemisor

Emite a nombre de varios RFC gestionando los CSD por API. **No se refleja en la
plataforma web.**

| Módulo | Contenido |
|---|---|
| `Certificados` | Alta, actualización, consulta por RFC, listado y borrado de CSD |
| `Cfdis` | Emisión CFDI 4.0, consulta y filtrado, detalle, status, descarga, acuse, envío por email, cancelación (con y sin motivo) |
| `Complementos` | Complemento de pago, donativos, concepto por cuenta de terceros |

---


## 🔄 Flujo recomendado

1. **Mi cuenta** → verifica tu perfil fiscal y sube tu CSD (o, en API-Lite, da de alta el CSD del RFC emisor).
2. **Clientes** → crea un receptor y guarda su `Id`.
3. **Productos ó servicios** → crea un producto (opcional; también puedes mandar los conceptos inline).
4. **CFDI 4.0** → emite la factura y guarda el `Id` que devuelve.
5. **Detalle de Cfdi** / **CFDI Status SAT** → consulta el comprobante timbrado.
6. **Descargar PDF / XML** → obtén los archivos (respuesta en Base64).
7. **Enviar Cfdi por email** → opcional.
8. **Cancelar Cfdi** → opcional; requiere motivo de cancelación (y folio sustituto si aplica).

---

## 🧠 Notas importantes

- Los `Id` que traen las requests de ejemplo (clientes, CFDI, productos) son de una cuenta
  de pruebas: **reemplázalos por los tuyos**. Muchas requests dependen de datos creados antes.
- Varias operaciones requieren datos válidos del SAT (RFC existente, código postal, régimen
  fiscal y uso de CFDI compatibles). En sandbox usa `EKU9003173C9`.
- Las descargas (PDF, XML, acuse) devuelven el archivo **en Base64** dentro del JSON, no un binario.
- En producción, cada emisión timbra un CFDI real y consume folios. Verifica el entorno
  seleccionado antes de ejecutar un POST.
- Duplica las requests para tus pruebas en lugar de editar las originales; así el `git diff`
  se mantiene limpio.

---

## 🛠️ Requisitos

- [Postman](https://www.postman.com/downloads/) o [Bruno](https://www.usebruno.com/downloads)
- Cuenta activa en Facturama (sandbox o producción)

---

## 🤝 Contribuciones

Las contribuciones son bienvenidas. Abre un issue o un pull request.

Al aportar una request nueva:

1. Agrégala en **ambas** colecciones (`postman/` y `bruno/`) con el mismo nombre y módulo.
2. Usa siempre `{{SERVER}}`, `{{UserName}}` y `{{Password}}`; nunca hardcodees credenciales ni hosts.
3. Documenta la request (campo `docs` en Bruno / `description` en Postman).
4. Exporta la colección de Postman como **Collection v2.1** para no romper el diff.

---

## 📄 Licencia

MIT License

---

## ⭐ Soporte

Si este proyecto te es útil, considera darle una estrella ⭐ al repositorio.
