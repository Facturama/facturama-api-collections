# 🚀 Facturama API Collections — Postman & Bruno

> [!NOTE]
> Este documento también está disponible en [Español](README.md).

API collections for [Facturama](https://api.facturama.mx/) to test, understand and
integrate the Mexican e-invoicing (CFDI 4.0) endpoints without writing a single line of code.

The same set of requests ships in **two formats**:

| Client | Folder | Format |
|---|---|---|
| **Postman** | [postman/](postman/) | Collection v2.1 (`FacturamaAPI.postman_collection_v2.1.json`) + environments |
| **Bruno** | [bruno/](bruno/) | OpenCollection 1.0 (folders + `.yml`, Git-friendly) |

Full API guide: <https://apisandbox.facturama.mx/guias>

---

## ⚡ Quick Start

### Postman

1. Clone or download this repository.
2. **Import** → `postman/FacturamaAPI.postman_collection_v2.1.json`.
3. **Import** → both environments from `postman/Environments/`:
   - `Facturama API Test.postman_environment.json`
   - `Facturama API Prod.postman_environment.json`
4. Select the **Facturama API Test** environment and fill in `UserName` and `Password`.
5. Run any request.

### Bruno

1. Clone or download this repository.
2. **Open Collection** → pick the `bruno/` folder (the one containing `opencollection.yml`).
   There is no file to import: Bruno opens the folder as-is and your changes stay in Git.
3. Select the **Sandbox** environment and fill in `Password` and the `UserName`.
4. Run any request.

> Your Bruno version must support the **OpenCollection 1.0** format. If the requests
> don't show up after opening the folder, update Bruno to the latest version.

✅ In a few minutes you can run real operations against the API.

---

## 🔐 Authentication

Every request uses **HTTP Basic Auth** with your Facturama account credentials, plus a
`User-Agent` header carrying your username:

| Variable | Purpose |
|---|---|
| `SERVER` | API base host (depends on the environment) |
| `UserName` | Account user → Basic Auth + `User-Agent` header |
| `Password` | Account password → Basic Auth |

In Postman auth is defined **per request** (not inherited from the collection); in Bruno
folders use `auth: inherit`. In both cases the client's automatic `User-Agent` is disabled
so the custom one is sent.

> [!WARNING]
> Never commit real credentials. In Bruno, `Password` is already flagged as `secret` and is
> not serialized into the environment file. In Postman, mark the value as **secret** in the
> environment before saving or exporting it.

---

## 🌍 Environments

| Environment | Postman | Bruno | `SERVER` |
|---|---|---|---|
| Testing | `Facturama API Test` | `Sandbox` | `https://apisandbox.facturama.mx` |
| Production | `Facturama API Prod` | `Producción` | `https://api.facturama.mx` |

Names differ between clients, but the variables and hosts are identical. Select the
environment **before** running any request.

### Test account

- Create an account in the [Sandbox](https://dev.facturama.mx/api/registro) environment.
- For the **Web API**, set up your tax profile with the test RFC **`EKU9003173C9`**
  ([guide](https://apisandbox.facturama.mx/guias/perfil-fiscal)).
- Test digital seal certificates (CSD): [facturama-CSD-prueba](https://apisandbox.facturama.mx/guias/conocimientos/sellos-digitales-pruebas).

---

## 🧩 Structure

Both collections are split into the two Facturama API families:

### `API Web` — single issuer

Operates on your own account (the configured tax profile) and **everything is reflected in
the Facturama web platform**.

| Module | Contents |
|---|---|
| `Clientes` | Create, edit, delete, list, filter by RFC, RFC validation and status |
| `Productos ó servicios` | Create/edit/delete, paginated lists, products with VAT, IEPS, ISR, withholdings, property tax account |
| `CFDI 4.0` | Income, lease, credit notes, global invoice, general public, foreign currency, IEPS, VAT exempt / 0%, border stimulus, foreign customer, lookup and filtering, SAT status, PDF/XML/acknowledgement download, email delivery, cancellation |
| `Complementos` | Payments (multiple variants), Carta Porte 3.1 (road, air, sea, rail, international), Foreign Trade, Payroll, INE, IEDU, Donations, Hydrocarbons, Tax Legends, Public Notaries |
| `Retenciones` | Withholdings v2 and technology platforms, detail, filters, download and cancellation |
| `Mi cuenta` | Billing data, `TaxEntityInfo`, CSD/FIEL upload, logo, series, issuing places, subscription |
| `Catalogos` | SAT catalogs: general, Carta Porte and Payroll |

### `API-Lite` — multi-issuer

Issues on behalf of multiple RFCs, managing CSDs through the API. **Not reflected in the
web platform.**

| Module | Contents |
|---|---|
| `Certificados` | Upload, update, get by RFC, list and delete CSDs |
| `Cfdis` | CFDI 4.0 issuance, lookup and filtering, detail, status, download, acknowledgement, email delivery, cancellation (with and without reason) |
| `Complementos` | Payment complement, donations, third-party account concept |

---

## 🔄 Recommended flow

1. **Mi cuenta** → check your tax profile and upload your CSD (or, in API-Lite, register the issuing RFC's CSD).
2. **Clientes** → create a receiver and keep its `Id`.
3. **Productos ó servicios** → create a product (optional; you can also send line items inline).
4. **CFDI 4.0** → issue the invoice and keep the returned `Id`.
5. **Detalle de Cfdi** / **CFDI Status SAT** → look up the stamped document.
6. **Descargar PDF / XML** → get the files (Base64 response).
7. **Enviar Cfdi por email** → optional.
8. **Cancelar Cfdi** → optional; requires a cancellation reason (and a replacement folio when applicable).

---

## 🧠 Important notes

- The `Id` values in the sample requests (clients, CFDIs, products) belong to a test
  account: **replace them with your own**. Many requests depend on previously created data.
- Several operations require valid SAT data (existing RFC, postal code, compatible tax
  regime and CFDI use). In sandbox, use `EKU9003173C9`.
- Downloads (PDF, XML, acknowledgement) return the file **Base64-encoded** inside the JSON,
  not as a binary body.
- In production every issuance stamps a real CFDI and consumes folios. Double-check the
  selected environment before running a POST.
- Duplicate requests for your own tests instead of editing the originals; that keeps the
  `git diff` clean.

---

## 🛠️ Requirements

- [Postman](https://www.postman.com/downloads/) or [Bruno](https://www.usebruno.com/downloads)
- An active Facturama account (sandbox or production)

---

## 🤝 Contributing

Contributions are welcome. Open an issue or a pull request.

When adding a new request:

1. Add it to **both** collections (`postman/` and `bruno/`) with the same name and module.
2. Always use `{{SERVER}}`, `{{UserName}}` and `{{Password}}`; never hardcode credentials or hosts.
3. Document the request (`docs` field in Bruno / `description` in Postman).
4. Export the Postman collection as **Collection v2.1** so the diff stays readable.

---

## 📄 License

MIT License

---

## ⭐ Support

If this project is useful to you, consider giving the repository a star ⭐.
