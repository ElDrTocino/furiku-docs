# 📘 Furiku Docs

**Furiku Docs** es la documentación oficial para la plataforma SaaS **Furiku**, especializada en la gestión moderna de cafeterías, restaurantes y dark kitchens.  
Este repositorio contiene documentación técnica estructurada, mantenida con [Docusaurus 2](https://docusaurus.io/) y escrita en Markdown + TypeScript.

---

## 🚀 ¿Qué contiene esta documentación?

- 🧱 Arquitectura del backend (modular, multitenant-ready)
- 🔐 Autenticación con JWT y roles
- 🏢 Separación de datos por tenant
- 📦 Soft delete y auditoría embebida
- 👤 Módulo de usuarios
- 🏪 Módulo de negocios
- 📄 Buenas prácticas de DTOs y API REST

---

## 📂 Estructura del proyecto

```
furiku-docs/
├── docs/                # Archivos Markdown por módulo o feature
├── sidebars.ts          # Configuración del menú lateral
├── docusaurus.config.ts # Configuración principal del sitio
├── src/
│   └── css/custom.css   # Estilos personalizados
├── static/              # Archivos estáticos (logos, favicon, etc.)
├── tsconfig.json        # Configuración TypeScript
└── README.md
```

---

## ▶️ ¿Cómo correrlo en local?

```bash
npm install
npm run start
```

Esto abrirá la documentación en:  
📍 `http://localhost:3000`

---

## ✍️ ¿Cómo agregar una nueva página?

1. Crea un archivo `.md` dentro de `docs/`
2. Asegúrate de agregarlo en `sidebars.ts`
3. Reinicia el servidor con `npm run start`

---

## 📦 ¿Cómo hacer deploy?

Puedes desplegar este proyecto en:

- **GitHub Pages**
- **Vercel**
- **Netlify**

Consulta la [documentación oficial](https://docusaurus.io/docs/deployment) para más detalles.
