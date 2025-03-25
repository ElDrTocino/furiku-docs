---
id: backend-architecture
title: 🧱 Arquitectura General del Backend
sidebar_label: Arquitectura Backend
---

La arquitectura del backend de Furiku está diseñada para ser limpia, modular, escalable y multitenant desde el inicio. Utiliza tecnologías modernas como **Quarkus**, **JWT**, y **Hibernate Panache**, asegurando alto rendimiento y una base sólida para evolución futura hacia microservicios.

---

## 🧰 Stack tecnológico

| Componente     | Tecnología         |
|----------------|--------------------|
| Lenguaje       | Java (17+)         |
| Framework      | Quarkus            |
| ORM            | Hibernate Panache  |
| Seguridad      | Quarkus Security (JWT) |
| Base de datos  | MySQL              |
| Build system   | Gradle             |
| Internacionalización | Propiedades + helper |
| API            | RESTful con JAX-RS |
| Documentación  | Docusaurus         |

---

## 🗂️ Estructura de paquetes

La arquitectura está dividida por dominio de negocio:

    org.furiku
    ├── user
    │   ├── controller
    │   ├── dto
    │   ├── model
    │   ├── repository
    │   └── service
    ├── business
    ├── auth
    ├── common
    │   ├── audit
    │   ├── i18n
    │   ├── repository
    │   └── security
    └── ...

Cada módulo es autocontenible y representa un **bounded context**, lo cual favorece la futura migración a microservicios.

---

## 🧩 Principios clave de diseño

- **Modularidad:** cada feature está en su propio paquete raíz (`user`, `business`, etc.)
- **Separación de capas:** cada módulo tiene sus capas `model`, `service`, `controller`, `dto`, `repository`
- **Internacionalización integrada:** todos los mensajes usan claves `MessageKeys` con `.properties`
- **Seguridad centralizada:** control de acceso con JWT y `SecurityIdentity`
- **Multitenancy desde el diseño:** cada entidad tiene `tenantId` como identificador contextual

---

## 🧱 Capas principales

| Capa        | Descripción |
|-------------|-------------|
| **Model**       | Entidades JPA (`@Entity`) con auditoría y soft delete |
| **Repository**  | Interfaces que extienden `SoftDeletableRepository<T, ID>` para filtrar automáticamente `deleted = false` |
| **Service**     | Contiene la lógica de negocio y validaciones de seguridad contextual |
| **DTO**         | Objetos usados para entrada/salida de la API |
| **Controller**  | Define rutas expuestas mediante JAX-RS (`@Path`, `@GET`, `@POST`) |
| **Common**      | Código compartido (seguridad, helpers, internacionalización, auditoría) |

---

## 🔐 Seguridad y roles

La autenticación se maneja mediante JWT, usando `quarkus-security`. Los roles disponibles (`OWNER`, `MANAGER`, `STAFF`, `CASHIER`) se aplican con `@RolesAllowed` en cada controlador.  
El contexto actual se accede vía:

    @Inject CurrentUser currentUser;

---

## 🌍 Internacionalización (i18n)

Todos los errores, respuestas y validaciones utilizan un sistema de internacionalización con:

- Clases `MessageKeys` centralizadas
- Archivos `.properties` (`messages.properties`, `messages_es.properties`)
- Inyección de `MessageProvider` para obtener mensajes por clave

    messageProvider.get(MessageKeys.USER_ALREADY_EXISTS);

Esto asegura que Furiku esté preparado para multiidioma desde el principio.

---

## 🕵️ Auditoría y Soft Delete

Cada entidad importante hereda de `AuditableEntity` que contiene:

- `createdAt`, `updatedAt`, `deletedAt`, `deletedBy`, `isDeleted`
- Campo embebido `AuditMetadata` con: `performedBy`, `performedRole`, `performedTenant`, `performedAt`

Esto permite trazabilidad completa de todas las acciones.

---

## 📦 Multitenancy

Toda operación en Furiku está contextualizada por `tenantId`, que representa un negocio:

- Cada usuario pertenece a un `tenantId`
- Cada entidad contiene un campo `tenantId`
- Toda validación compara `currentUser.getTenantId()` contra el recurso objetivo

---

## 🚀 Preparado para microservicios

Aunque Furiku es un backend monolítico modular por ahora, su estructura está pensada para una futura separación en servicios independientes.  
Cada módulo cumple con los principios de un microservicio:

- Aislado por dominio
- Bajo acoplamiento
- Contratos bien definidos vía DTO
- Validaciones propias
- Independencia de repositorio y lógica

---

## 🧠 Buenas prácticas implementadas

- Código limpio con Lombok (`@Builder`, `@Getter`, `@RequiredArgsConstructor`, etc.)
- Sin uso directo de `@Inject` innecesario
- Validación contextual (`CurrentUser`) para evitar acceso cruzado
- Logs relevantes con `@Slf4j`
- Tests (a implementar) por módulo

---

## 🧩 Conclusión

La arquitectura de backend de Furiku está lista para escalar, auditar y mantener fácilmente.  
Modular, organizada y profesional, permite que cualquier nuevo desarrollador entienda rápidamente dónde vive cada pieza del sistema.

> 🧱 Este backend está hecho para crecer. Cada línea de código sigue el principio: “escalabilidad desde el primer commit”.
