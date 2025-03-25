---
id: multitenancy
title: 🏢 Arquitectura Multitenant
sidebar_label: Arquitectura Multitenant
---

Furiku fue diseñado desde el inicio para ser una **plataforma SaaS multitenant**, lo cual significa que múltiples negocios (tenants) pueden compartir la misma aplicación sin interferir entre sí.  
Cada negocio tiene sus propios datos, usuarios, configuraciones y operaciones totalmente separadas del resto.

---

## 🧠 ¿Qué es multitenancy?

Multitenancy es una arquitectura en la cual una sola instancia de la aplicación sirve a múltiples "clientes" (tenants), manteniendo **aislamiento lógico** entre ellos.

En Furiku, un tenant puede ser:

- Una cafetería
- Un restaurante
- Una dark kitchen
- Un food truck
- Cualquier negocio independiente

---

## 🧱 Estrategia de multitenancy aplicada

Furiku aplica multitenancy mediante **segregación lógica de datos**:

- Cada entidad relacionada al negocio (usuarios, productos, órdenes, facturas) contiene un campo:

        tenantId: String

- Este campo representa el identificador único del negocio.

---

## 🔐 Validaciones contextuales

El backend protege todos los recursos con validaciones estrictas de `tenantId`.  
Se accede al tenant actual desde el JWT mediante el helper:

    currentUser.getTenantId();

### 🔒 Reglas generales:

| Acción | Requiere validación de tenant |
|--------|-------------------------------|
| Consultar usuarios | ✅ |
| Crear productos | ✅ |
| Eliminar órdenes | ✅ |
| Consultar negocio | ✅ |
| Ver estadísticas | ✅ |
| Eliminar otro negocio | ❌ Rechazado (solo OWNER del mismo tenant) |

---

## 🔍 ¿Dónde se valida el tenant?

En los **servicios** (no en los controladores) mediante comparaciones como:
```java
    if (!entity.getTenantId().equals(currentUser.getTenantId())) {
        throw new WebApplicationException(...);
    }
```

Y también mediante filtrados en los repositorios:

```java
    find("tenantId = ?1 and deleted = false", tenantId)
```

---

## 🧰 Patrón de diseño aplicado

La multitenancy está integrada en:

- El modelo (`tenantId` como campo obligatorio)
- El repositorio (consultas filtradas)
- El token (JWT contiene el tenant del usuario)
- El servicio (validaciones de acceso cruzado)
- La auditoría (cada acción registra `performedTenant`)

---

## 🧪 Ejemplo: Crear un producto

Un `MANAGER` de "cafe-del-mar" intenta crear un producto.

1. El token JWT contiene:

        "tenantId": "cafe-del-mar"

2. Al guardar el producto, el sistema fuerza:

        product.setTenantId(currentUser.getTenantId());

3. Nadie de otro tenant podrá ver o editar ese producto.

---

## 🧪 Ejemplo: Acceso indebido

Un `OWNER` de "taco-place" intenta acceder a usuarios de "cafe-del-mar":

- Aunque tenga rol `OWNER`, el sistema compara `currentUser.getTenantId()` con el `tenantId` del recurso solicitado.
- Si no coinciden, se lanza:

        HTTP 403 Forbidden — Access denied to resource in another tenant.

---

## 🧩 Ventajas del enfoque actual

| Ventaja | Descripción |
|---------|-------------|
| 🔐 Seguridad | Aislamiento total por tenant |
| ♻️ Escalabilidad | Agregar más tenants no afecta a otros |
| 📦 Simplicidad | Una sola base de datos y una sola app |
| 🔍 Auditoría clara | Cada acción incluye tenant de origen |

---

## ⚠️ Riesgos a mitigar

- Nunca confiar solo en el frontend para enviar `tenantId`
- Siempre tomar el tenant desde el token JWT
- No permitir búsquedas globales sin validación
- Evitar relaciones entre entidades de distintos tenants

---

## 🚀 Evolución futura

Aunque hoy Furiku usa **segregación lógica en una misma base de datos**, está listo para migrar a:

- **Bases de datos separadas por tenant**
- **Schemas por tenant**
- **Microservicios con aislación completa**

Esto permitiría aún mayor independencia y control de escalabilidad.

---

## ✅ Buenas prácticas implementadas

- Cada entidad importante incluye `tenantId`
- Todo acceso pasa por `CurrentUser`
- Solo `OWNER` puede eliminar o modificar el negocio
- Las queries usan `findByIdActive(tenantId)` o filtros seguros
- Las respuestas y errores están internacionalizadas

---

## 🧩 Conclusión

La arquitectura multitenant de Furiku permite ofrecer un SaaS profesional donde múltiples negocios operan sin interferencias.  
Es segura, limpia y extensible hacia soluciones más avanzadas si el sistema crece.

> 🏢 Un tenant es una frontera lógica. En Furiku, esa frontera es inquebrantable.
