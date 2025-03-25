---
id: business-module
title: 🏪 Módulo de Negocios (Tenants)
sidebar_label: Módulo de Negocios
---

El módulo de negocios (o tenants) en Furiku representa a cada cliente del sistema: cafeterías, restaurantes pequeños, dark kitchens, food trucks, etc.

Cada negocio es independiente en datos y usuarios, y funciona como **unidad aislada** dentro de la arquitectura multitenant.

---

## 🧩 ¿Qué es un `Business`?

Una entidad `Business` representa a un tenant único en Furiku. Todo usuario, producto, orden y operación está relacionada con un `tenantId`.

---

## 🧾 Campos principales en `Business`

| Campo       | Descripción |
|-------------|-------------|
| `tenantId`  | Identificador único del negocio (usado como clave primaria) |
| `name`      | Nombre del negocio |
| `rfc`       | Registro Federal de Contribuyentes (para facturación SAT) |
| `address`   | Dirección del local |
| `plan`      | Tipo de plan contratado (`FREE`, `PREMIUM`, etc.) |
| `audit`     | Información embebida sobre quién y cuándo se creó/modificó |
| `isDeleted` | Bandera de eliminación lógica (soft delete) |

---

## 📦 ¿Qué representa un `tenant`?

En Furiku, un tenant es **una unidad de negocio completamente separada**, con su propio:

- Conjunto de usuarios
- Inventario
- Órdenes
- Facturación
- Reportes

---

## 🛠 ¿Cómo se crea un negocio?

```json
    POST /api/businesses

    {
        "tenantId": "cafe-centro",
        "name": "Café Centro",
        "rfc": "CAC010203AB9",
        "address": "Av. Principal 123",
        "plan": "FREE"
    }
```  



> ⚠️ El `tenantId` debe ser único. Si ya existe, se devolverá un error `BUSINESS_ALREADY_EXISTS`.

---

## 🧪 Ejemplo de respuesta

```json

    {
        "tenantId": "cafe-centro",
        "name": "Café Centro",
        "plan": "FREE"
    }
```    

---

## 🔍 Obtener negocio por ID

```java

    GET /api/businesses/{tenantId}
```    

Este endpoint retorna el negocio **si no ha sido eliminado** (`isDeleted = false`).

---

## ✏️ Editar información del negocio

```json  

    PUT /api/businesses/{tenantId}

    {
        "name": "Nuevo Café Centro",
        "rfc": "CAC010203AB9",
        "address": "Av. Nueva 456",
        "plan": "PREMIUM"
    }
```    

> Solo usuarios con rol `OWNER` pueden editar el negocio, y deben pertenecer al mismo tenant.

---

## ❌ Soft delete de negocio

```java

    DELETE /api/businesses/{tenantId}
```

Marca el negocio como eliminado (`isDeleted = true`) y registra:

- `deletedAt`
- `deletedBy`

El negocio ya no podrá ser consultado ni modificado. Esta acción **solo puede realizarla el OWNER del tenant**.

---

## 🛡️ Seguridad y contexto

Todas las operaciones sobre negocios están protegidas por:

- `@RolesAllowed("OWNER")` en endpoints sensibles
- Validación del `tenantId` actual contra el `CurrentUser`

---

## 🧠 Validaciones importantes

| Validación                  | Resultado |
|----------------------------|-----------|
| Tenant ya existe           | ❌ Error `409 Conflict` con mensaje internacionalizado |
| Tenant no encontrado       | ❌ Error `404 Not Found` |
| Tenant eliminado           | ❌ No se permite modificar ni consultar |
| Modificación por otro rol  | ❌ Error `403 Forbidden` |
| Acceso cruzado de tenants  | ❌ Rechazado mediante `CurrentUser.getTenantId()` |

---

## ✅ Buenas prácticas

- Siempre valida que `CurrentUser.getTenantId()` coincida con el tenant objetivo  
- Utiliza `AuditHelper.generate()` en cada operación que modifique el negocio  
- Usa `businessRepository.findByIdActive(tenantId)` para evitar errores con eliminados  
- Solo permite edición y eliminación a `OWNER` del mismo tenant  
- Si vas a mostrar negocios, asegúrate de filtrarlos por `deleted = false`

---

## 🚀 Futuras mejoras

- Historial de cambios del negocio (con Hibernate Envers)  
- Límite de funcionalidades según `plan` (`FREE`, `PREMIUM`)  
- Panel administrativo de planes y facturación  
- Asociación de múltiples sucursales a un mismo negocio  
- Activación/desactivación temporal por falta de pago

---

## 🧩 Conclusión

El módulo `Business` es la base de la arquitectura multitenant de Furiku.  
Cada operación en el sistema depende del tenant activo, asegurando:

- Separación de datos
- Seguridad
- Escalabilidad
- Independencia entre clientes

> 🏗️ Todo parte del `tenantId`, y este módulo garantiza que cada cafetería, dark kitchen o restaurante tenga su espacio propio dentro de Furiku.
