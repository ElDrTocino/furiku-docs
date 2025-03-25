---
id: soft-delete-audit
title: 🧾 Auditoría y Soft Delete
sidebar_label: Auditoría y Soft Delete
---

Furiku implementa una estrategia sólida de trazabilidad basada en una entidad abstracta `AuditableEntity` que proporciona control total sobre las acciones del sistema.

---

## 🧠 ¿Qué incluye `AuditableEntity`?

- `@Embedded AuditMetadata`: 
  - `performedBy`
  - `performedRole`
  - `performedTenant`
  - `performedAt`

- Timestamps automáticos:
  - `createdAt`
  - `updatedAt`

- Soporte para **soft delete**:
  - `isDeleted`
  - `deletedAt`
  - `deletedBy`

Todas las entidades críticas del sistema heredan de esta clase base para garantizar trazabilidad y limpieza de datos.

---

## 🔐 ¿Qué es Soft Delete?

Soft delete es una estrategia en la que una entidad no se elimina físicamente de la base de datos, sino que se marca como eliminada mediante los campos mencionados anteriormente.

Esto permite:

- Recuperar datos si es necesario  
- Mantener integridad histórica y auditoría completa  
- Filtrar fácilmente solo los registros “activos” (`isDeleted = false`)

---

## 📦 ¿Dónde se usa?

Actualmente, se implementa en:

- `User`
- `Business`

Y está diseñado para expandirse a:

- `Order`
- `Product`
- `Invoice`
- `InventoryChange`
- etc.

---

## 🛠 ¿Cómo se usa?

### 1. Hereda de `AuditableEntity`

    public class User extends AuditableEntity {
        // ...
    }

Esto asegura que la entidad tenga todos los campos necesarios sin repetir código.

---

### 2. Usa el repositorio base
```java
    public class UserRepository implements SoftDeletableRepository<User, Long> {
        // ya puedes usar findActive(), findByIdActive(), existsByFieldActive(), softDeleteById()
    }
```

Esto elimina la necesidad de repetir `deleted = false` en cada query.

---

### 3. Aplica auditoría al crear/editar

Utiliza `AuditHelper` para registrar automáticamente quién realizó la acción, en qué tenant y en qué momento:

    user.setAudit(auditHelper.generate());

---

### 4. Elimina con trazabilidad (soft delete)

    userRepository.softDeleteById(userId, currentUser.getEmail());

Esto marcará:

- `isDeleted = true`
- `deletedAt = Instant.now()`
- `deletedBy = currentUser.getEmail()`

Y la entidad será excluida de consultas estándar automáticamente.

---

## 🧪 Métodos disponibles en `SoftDeletableRepository`

| Método | Descripción |
|--------|-------------|
| `findActive()` | Devuelve todos los registros activos |
| `findByIdActive(ID)` | Busca por ID solo si no está eliminado |
| `findByFieldActive("email", value)` | Consulta por campo filtrando los eliminados |
| `existsByFieldActive("email", value)` | Verifica existencia excluyendo eliminados |
| `softDeleteById(ID, deletedBy)` | Marca como eliminado con auditoría |

---

## 🚨 Buenas prácticas

- Siempre usa `findActive()` o `findByIdActive()` en servicios  
- No muestres datos eliminados a usuarios normales  
- Solo `OWNER` puede eliminar registros sensibles  
- Usa `auditHelper.generate()` en cada creación o actualización  
- No permitas edición de registros eliminados

---

## ✅ Beneficios

| Ventaja | Descripción |
|---------|-------------|
| 🔐 Seguridad | Nunca pierdes información crítica |
| 🔁 Reversibilidad | Puedes restaurar si es necesario |
| 📊 Métricas limpias | Puedes excluir eliminados en reportes |
| 🧠 Escalable | Se aplica fácilmente a cualquier entidad del sistema |

---

## 🧩 Extensiones futuras

- Filtros automáticos globales en Panache  
- Historial completo con Hibernate Envers  
- Vista de auditoría por entidad en el panel de administración

> ✨ Esta estrategia asegura que Furiku sea escalable, auditable y profesional desde su núcleo.
