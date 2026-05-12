# [Nombre del Proyecto]

Descripción en 2-3 oraciones. Qué hace, para quién y
cuál es el stack principal.

---

## Stack

- Next.js 15 + TypeScript
- TanStack Query — server state
- Zustand — UI state
- Supabase — base de datos y auth
- [otras librerías relevantes]

---

## Requisitos previos

- Node.js 20+
- [Otros requisitos]

---

## Instalación

git clone [url]
cd [proyecto]
npm install
cp .env.example .env.local
npm run dev

---

## Variables de entorno

NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
[otras variables necesarias]

---

## Estructura del proyecto

src/
app/ — páginas y rutas de Next.js
components/ — componentes reutilizables
hooks/ — custom hooks (queries, mutations)
stores/ — Zustand stores
services/ — data access layer
types/ — tipos TypeScript compartidos
docs/ — arquitectura y decisiones de diseño

---

## Documentación de arquitectura

Ver /docs para el diseño del sistema, decisiones
arquitecturales y contratos de API.
