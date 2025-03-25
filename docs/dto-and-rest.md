---
id: dto-and-rest
title: 📦 DTOs y Buenas Prácticas de API REST
sidebar_label: DTOs y API REST
---

Furiku sigue una estructura clara y consistente para el uso de **DTOs (Data Transfer Objects)** y el diseño de sus **API REST**, asegurando código limpio, mantenible y predecible para los desarrolladores.

---

## 📦 ¿Qué es un DTO?

Un DTO (Data Transfer Object) es un objeto que se utiliza para **transportar datos** entre capas del sistema, especialmente entre:

- Controladores y servicios
- Servicios y clientes (API REST)
- Operaciones de entrada (input) y salida (output)

> 🚫 Los DTOs **no deben contener lógica de negocio**, solo datos.

---

## 🧱 Convención de nombres en Furiku

Furiku utiliza el sufijo `DTO` al final de cada clase DTO para mantener claridad inmediata en el código.

### Ejemplos:

- `UserRequestDTO`: datos enviados al registrar un usuario
- `UserResponseDTO`: datos devueltos al consultar un usuario
- `LoginRequestDTO`: datos de login
- `BusinessResponseDTO`: información del negocio al crearlo o consultarlo

> ✅ El nombre describe su propósito (`Request`, `Response`, `Update`, `Login`, etc.)

---

## 🗂️ Organización por módulo

Los DTOs están ubicados en cada paquete de dominio:

    org.furiku.user.dto
    org.furiku.business.dto
    org.furiku.order.dto
    org.furiku.product.dto

Esto permite que cada módulo sea autocontenible y escalable.

---

## 📥 Ejemplo: `UserRequestDTO`

```java
    package org.furiku.user.dto;

    import lombok.Data;
    import jakarta.validation.constraints.*;

    @Data
    public class UserRequestDTO {
        @NotBlank
        private String name;

        @Email
        @NotBlank
        private String email;

        @NotBlank
        private String password;

        private String role; // optional, defaults to STAFF

        @NotBlank
        private String tenantId;
    }
```      

---

## 📤 Ejemplo: `UserResponseDTO`

```java
    @Data
    @Builder
    public class UserResponseDTO {
        private String name;
        private String email;
        private String role;
        private String tenantId;
    }
```    

---

## 🌐 Buenas prácticas en el diseño de API REST

### ✔️ Usar nombres semánticos y plurales

- `/api/users`  
- `/api/products`  
- `/api/orders/{id}`  

> Los endpoints deben representar recursos, no acciones (`/login` es una excepción válida).

---

### ✔️ Usar los métodos HTTP correctos

| Método | Uso común |
|--------|-----------|
| `GET`    | Obtener recursos |
| `POST`   | Crear nuevos recursos |
| `PUT`    | Actualizar completamente un recurso |
| `PATCH`  | Actualización parcial (si aplica) |
| `DELETE` | Eliminar (soft delete en Furiku) |

---

### ✔️ Separar claramente input y output

- `RequestDTO` para entrada
- `ResponseDTO` para salida

Evita usar entidades JPA (`@Entity`) directamente como respuesta o entrada.

---

### ✔️ Validaciones automáticas

Todos los DTOs de entrada deben usar anotaciones como:

    @NotBlank
    @Email
    @Min
    @Pattern

Esto reduce errores y evita lógica repetida en servicios.

---

### ✔️ Internamente consistente

Furiku mantiene la convención:

- Todos los controladores reciben y devuelven DTOs
- Los servicios trabajan con entidades
- Los mappers (si son necesarios) convierten entre DTO ↔ entidad

---

### ⚠️ Qué evitar

- ❌ Usar la entidad como payload de respuesta
- ❌ Devolver passwords o datos sensibles
- ❌ Mezclar campos innecesarios entre módulos
- ❌ Cambiar el nombre de campos entre request y response

---

## 🧩 Conclusión

Furiku mantiene una organización clara y consistente para sus DTOs y APIs REST, siguiendo los principios de:

- Claridad de intención
- Separación de capas
- Validación automática
- Diseño RESTful estándar

> 📦 Un buen diseño de DTOs y REST no solo facilita el desarrollo, sino que mejora la mantenibilidad, escalabilidad y experiencia para futuros colaboradores.
