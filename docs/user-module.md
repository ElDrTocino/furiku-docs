---
id: user-module
title: 👤 Módulo de Usuarios
sidebar_label: Módulo de Usuarios
---

El módulo de usuarios en Furiku administra a todas las personas que interactúan con el sistema: dueños, gerentes, empleados, etc.  
Está diseñado con roles claros, validaciones estrictas y control de acceso basado en JWT y `SecurityIdentity`.

---

## 👥 ¿Qué es un usuario en Furiku?

Un `User` representa a una persona que puede acceder al sistema. Cada usuario:

- Pertenece a un `tenant` (negocio, cafetería, restaurante, etc.)
- Tiene un `rol` que determina qué puede hacer
- Tiene un correo electrónico único y una contraseña segura
- Puede ser soft-eliminado (no eliminado físicamente)
- Tiene trazabilidad completa (auditoría embebida)

---

## 🧾 Campos principales en la entidad `User`

| Campo         | Descripción |
|---------------|-------------|
| `id`          | Identificador único (auto generado) |
| `name`        | Nombre completo del usuario |
| `email`       | Email único y usado como login |
| `password`    | Contraseña cifrada con BCrypt |
| `role`        | Enum: `OWNER`, `MANAGER`, `STAFF`, `CASHIER` |
| `tenantId`    | Negocio al que pertenece |
| `isDeleted`   | Bandera booleana para soft delete |
| `audit`       | Información de auditoría embebida |

---

## 🔐 Roles disponibles

Furiku define 4 roles básicos para controlar el acceso:

| Rol       | Descripción |
|-----------|-------------|
| `OWNER`   | Dueño del negocio. Tiene acceso total al sistema, incluyendo gestión de usuarios y configuración del negocio. |
| `MANAGER` | Gerente del negocio. Puede gestionar productos, órdenes y ver reportes. Puede crear usuarios secundarios. |
| `STAFF`   | Empleado operativo. Accede solo a creación de órdenes y productos si se le permite. |
| `CASHIER` | Cajero. Solo puede visualizar y gestionar cobros y emisión de facturas. |

---

## 🔐 Permisos por rol

A continuación se muestra una tabla de acciones y los roles que pueden realizarlas:

| Acción                                | OWNER | MANAGER | STAFF | CASHIER |
|---------------------------------------|:-----:|:-------:|:-----:|:-------:|
| Crear usuarios                        | ✅    | ✅      | ❌    | ❌      |
| Editar perfil de usuario              | ✅    | ✅      | ✅    | ✅      |
| Eliminar (soft delete) usuarios       | ✅    | ❌      | ❌    | ❌      |
| Crear productos                       | ✅    | ✅      | ✅    | ❌      |
| Actualizar productos                  | ✅    | ✅      | ✅    | ❌      |
| Crear órdenes                         | ✅    | ✅      | ✅    | ✅      |
| Ver reportes                          | ✅    | ✅      | ❌    | ❌      |
| Emitir facturas                       | ✅    | ✅      | ❌    | ✅      |
| Eliminar negocio                      | ✅    | ❌      | ❌    | ❌      |
| Ver información del negocio (tenant) | ✅    | ✅      | ✅    | ✅      |

> ⚠️ Estas reglas son controladas en el backend usando `@RolesAllowed` y validaciones adicionales con `CurrentUser`.

---

## 🧪 Ejemplo 1: Registro de usuario

Cuando un usuario se registra, debe proporcionar:

- Nombre
- Email
- Contraseña
- Rol (opcional, por defecto: `STAFF`)
- `tenantId` (debe ser válido)

```json

    POST /api/users/register

    {
        "name": "Lucía Ramírez",
        "email": "lucia@ejemplo.com",
        "password": "segura123",
        "role": "STAFF",
        "tenantId": "cafeteria-el-sol"
    }
```

Si el `tenantId` no existe o el email ya está en uso, se devolverá un error con internacionalización.

---

## 🧪 Ejemplo 2: Inicio de sesión

```json
    POST /api/users/login

    {
        "email": "lucia@ejemplo.com",
        "password": "segura123"
    }
```

Si las credenciales son correctas y el usuario no ha sido eliminado, se devuelve un JWT y su información básica.

---

## 🧹 Soft delete de usuarios

Cuando un usuario es eliminado desde el panel administrativo, no se elimina físicamente de la base de datos. En su lugar:

- Se marca como `isDeleted = true`
- Se registra `deletedAt` y `deletedBy`
- Ya no puede iniciar sesión ni aparecer en listados activos

```java
    DELETE /api/users/{email}
```

Solo el `OWNER` puede realizar esta acción y únicamente dentro de su mismo `tenant`.

---

## 🛡️ Seguridad y autenticación

Furiku utiliza JWT para autenticar peticiones. Cada usuario que inicia sesión recibe un token firmado que debe incluir en el header:

```java
    Authorization: Bearer <token>
```    

Ese token contiene:

- `email`
- `role`
- `tenantId`
- Tiempo de expiración

La validación se realiza usando `quarkus-security` y `SecurityIdentity`.

---

## 🧠 Validaciones importantes

| Validación | Comportamiento |
|------------|----------------|
| Email duplicado | ❌ Registro rechazado con mensaje internacionalizado |
| Email inexistente en login | ❌ Respuesta con error 401 |
| Usuario eliminado | ❌ No puede iniciar sesión (error 403) |
| Rol inválido | ❌ Rechazado con mensaje `USER_ROLE_INVALID` |
| Tenant inexistente | ❌ Registro o acción rechazada |

Todos los mensajes están internacionalizados usando `MessageKeys`.

---

## ✅ Buenas prácticas

- Usa `CurrentUser` para validar el contexto actual de usuario (email, role, tenant)  
- Aplica `@RolesAllowed` en cada endpoint sensible para evitar acceso no autorizado  
- Llama a `auditHelper.generate()` en cada creación o modificación de usuario  
- Filtra usuarios usando `findActive()` o `findByIdActive()` para ignorar los eliminados  
- No permitas que un `MANAGER` o inferior cree o elimine usuarios sin verificación de tenant  

---

## 🚀 Extensiones futuras

- Verificación de email (en proceso)  
- Restauración de usuarios soft-eliminados  
- Gestión de permisos finos por módulo  
- Registro de sesiones activas y cierre remoto  
- Notificaciones para cambios de rol o eliminación

---

## 🧩 Conclusión

El módulo de usuarios de Furiku está diseñado para ser:

- Seguro  
- Escalable  
- Respetuoso del contexto multitenant  
- Fácil de auditar

Permite gestionar accesos de forma clara y profesional, alineado con las necesidades reales de cafeterías, dark kitchens y restaurantes pequeños.

> ✨ Este módulo es el núcleo del control de acceso en Furiku, y se expande a lo largo de toda la plataforma.
