---
id: authentication-jwt
title: 🔐 Autenticación y JWT
sidebar_label: Autenticación y JWT
---

Furiku utiliza JSON Web Tokens (JWT) como mecanismo principal de autenticación y autorización entre módulos, garantizando seguridad, escalabilidad y compatibilidad total con su arquitectura multitenant.

La implementación está basada en `quarkus-security` y no requiere un `JwtProvider` personalizado, manteniendo el código limpio y desacoplado.

---

## 🔄 ¿Cómo funciona el flujo de autenticación?

1. Un usuario se registra o inicia sesión  
2. Si las credenciales son válidas, el backend genera un token JWT  
3. El token es enviado al cliente en la respuesta  
4. El cliente lo incluye en todas las peticiones siguientes en el header:  

```java
        Authorization: Bearer <jwt-token>
```

5. El backend valida y extrae el contenido del token en cada endpoint protegido

---

## 🛠 Estructura del JWT en Furiku

El token JWT contiene:

| Campo        | Descripción |
|--------------|-------------|
| `sub`        | Email del usuario |
| `role`       | Rol del usuario (`OWNER`, `MANAGER`, etc.) |
| `tenantId`   | Negocio al que pertenece el usuario |
| `iat`        | Tiempo de emisión |
| `exp`        | Tiempo de expiración (por defecto: 24h) |

Ejemplo decodificado:

```java

    {
      "sub": "ana@cafe.com",
      "role": "MANAGER",
      "tenantId": "cafe-del-sol",
      "iat": 1711200000,
      "exp": 1711286400
    }
```    

---

## 🔐 ¿Cómo se protege un endpoint?

Furiku utiliza `@RolesAllowed(...)` para restringir acciones:

```java

    @GET
    @Path("/api/users")
    @RolesAllowed({"OWNER", "MANAGER"})
    public List<User> getUsers() { ... }
```    

También puede validarse el tenant con `CurrentUser.getTenantId()` para evitar accesos cruzados entre negocios.

---

## 🧠 ¿Cómo se obtiene el usuario actual?

A través del helper `CurrentUser`, que se apoya en `SecurityIdentity` de Quarkus:

```java

    currentUser.getEmail();
    currentUser.getRole();
    currentUser.getTenantId();
```    

Esto permite usar los datos del token sin tener que volver a consultar la base de datos.

---

## 🧪 Ejemplo de inicio de sesión

    POST /api/users/login

```json

    {
        "email": "ana@cafe.com",
        "password": "miClave123"
    }
```    
Respuesta exitosa:

```json
    {
      "name": "Ana Gómez",
      "email": "ana@cafe.com",
      "role": "MANAGER",
      "token": "<jwt-token>"
    }
```    

---

## 📥 ¿Cómo enviar el token desde el frontend?

En todas las peticiones protegidas:

```java

    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Ejemplo usando `fetch`:

```java

    fetch("/api/products", {
      method: "GET",
      headers: {
        "Authorization": "Bearer " + token
      }
    });
```    

---

## 🚨 Validaciones importantes

| Validación                         | Comportamiento |
|------------------------------------|----------------|
| Token ausente o malformado         | ❌ 401 Unauthorized |
| Rol insuficiente (`@RolesAllowed`) | ❌ 403 Forbidden |
| Acceso a otro tenant               | ❌ 403 Forbidden |
| Usuario eliminado (`isDeleted`)    | ❌ 403 Forbidden (en login) |
| Expiración del token               | ❌ 401 Unauthorized |

---

## ✅ Buenas prácticas

- Nunca guardar el token en `localStorage` (usa `HttpOnly Cookie` o `sessionStorage`)  
- El backend debe validar el `tenantId` con el recurso solicitado  
- Evita almacenar datos sensibles dentro del token  
- Regenera el token tras un cambio de rol o negocio

---

## 🔧 Configuración base en `application.properties`

    quarkus.jwt.enabled=true
    mp.jwt.verify.publickey.location=META-INF/resources/publicKey.pem
    quarkus.http.auth.permission.authenticated.paths=/api/*
    quarkus.http.auth.permission.authenticated.policy=authenticated

> Puedes usar `RS256` con un par de llaves generadas con OpenSSL, o `HS256` si prefieres clave secreta.

---

## 🧩 Conclusión

Furiku implementa JWT de forma limpia, usando `quarkus-security`, sin providers personalizados ni lógica innecesaria.  
Esto permite:

- Rutas protegidas por rol  
- Contexto de usuario accesible en cualquier parte del backend  
- Alto rendimiento y bajo acoplamiento

> 🔐 JWT es el corazón del control de acceso de Furiku. Toda acción parte de una identidad verificada.
