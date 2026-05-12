# [Nombre del Proyecto]

[Descripción en 2-3 oraciones. Qué hace la app, para quién es
y cuál es el flujo principal. Ejemplo: "Dashboard de gestión de
proyectos para equipos pequeños. Permite crear proyectos, asignar
tareas y hacer seguimiento del progreso en tiempo real."]

---

## Stack

- **Next.js [versión]** + TypeScript — framework principal
- **TanStack Query** — server state y caché
- **Zustand** — UI state
- **Supabase** — base de datos y autenticación
- **React Hook Form + Zod** — formularios y validación
- **[otras librerías relevantes]**

---

## Requisitos previos

- Node.js 20+
- npm 10+ / pnpm 9+
- Cuenta de Supabase (o instancia local)

---

## Instalación

```bash
git clone [url-del-repo]
cd [nombre-del-proyecto]
npm install
cp .env.example .env.local
```

Rellena las variables de entorno en `.env.local` (ver sección abajo)
y luego:

```bash
npm run dev
```

La app estará disponible en `http://localhost:3000`.

---

## Variables de entorno

```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=

# [Otras variables necesarias]
# NEXT_PUBLIC_[NOMBRE]=
```

> Las variables con `NEXT_PUBLIC_` son visibles en el cliente.
> Nunca pongas secrets sin ese prefijo en variables públicas.

---

## Scripts disponibles

```bash
npm run dev        # servidor de desarrollo
npm run build      # build de producción
npm run start      # servidor de producción
npm run lint       # ESLint
npm run typecheck  # TypeScript sin emitir archivos
```

---

## Estructura del proyecto

```
src/
  app/              — páginas y rutas (Next.js App Router)
  components/
    ui/             — componentes base reutilizables (Button, Input, Modal...)
    [feature]/      — componentes específicos de cada feature
  hooks/
    queries/        — TanStack Query hooks (fetching y mutations)
  stores/           — Zustand stores (solo UI state)
  services/         — data access layer (llamadas HTTP)
  types/            — tipos TypeScript compartidos
  lib/              — utilidades y configuración (api client, helpers)
docs/               — arquitectura y decisiones de diseño
```

---

## Documentación de arquitectura

El diseño del sistema, las decisiones arquitecturales y los
contratos de API están documentados en [`/docs`](./docs/README.md).

Si quieres entender por qué algo está construido de cierta manera,
empieza por [`/docs/architecture.md`](./docs/architecture.md).

---

## Convenciones del proyecto

**Nombrado de archivos:** kebab-case para archivos, PascalCase para componentes.

**Imports:** absolutos desde `src/` con alias `@/`.

**Server state vs UI state:** TanStack Query para datos del servidor,
Zustand solo para estado efímero de la UI (modales, filtros, drafts).

**[Agrega aquí otras convenciones del equipo]**
