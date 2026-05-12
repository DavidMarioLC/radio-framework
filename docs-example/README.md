# Task App

App web de gestión de tareas personales con autenticación.
Cada usuario puede crear, completar y eliminar sus propias tareas,
organizadas por estado: todas, pendientes y completadas.

---

## Stack

- **Next.js 15** + TypeScript — framework principal
- **TanStack Query** — server state y caché
- **Zustand** — UI state (filtro activo, usuario autenticado)
- **Supabase** — base de datos y autenticación
- **React Hook Form + Zod** — formularios de login y registro

---

## Requisitos previos

- Node.js 20+
- npm 10+ / pnpm 9+
- Cuenta de Supabase (o instancia local)

---

## Instalación

```bash
git clone [url-del-repo]
cd task-app
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
  app/
    (auth)/
      login/        — página de login
      register/     — página de registro
    tasks/          — página principal de tareas
    api/
      auth/         — endpoints de autenticación
      tasks/        — endpoints de tareas
  components/
    ui/             — componentes base (Button, Input, Skeleton...)
    tasks/          — TaskItem, TaskList, TaskComposer, TaskFilterBar
  hooks/
    queries/        — useTasksQuery, useCreateTask, useToggleTask, useDeleteTask
  stores/           — useAuthStore (usuario + filtro activo)
  services/         — taskService, authService
  types/            — tipos TypeScript compartidos
  lib/              — api client, helpers
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
Zustand solo para estado efímero de la UI (filtro activo, usuario en sesión).

**Optimistic updates:** solo en acciones frecuentes y reversibles (crear tarea,
toggle completado). Eliminar espera confirmación del servidor. Ver ADR-002.
