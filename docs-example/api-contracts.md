# API Contracts — Task App

> El servidor se trata como caja negra.
> Este documento define qué le enviamos y qué esperamos recibir.

---

## Convenciones

| Elemento | Convención                                        |
| -------- | ------------------------------------------------- |
| Base URL | `/api`                                            |
| Auth     | httpOnly cookie `session` (manejada por Supabase) |
| Fechas   | ISO 8601: `2026-05-12T10:30:00Z`                  |
| IDs      | UUID v4                                           |
| Errores  | `{ error: string, field?: string }`               |

---

## Tipos base

```typescript
type Task = {
  id: string;
  title: string;
  description: string | null;
  completed: boolean;
  createdAt: string;
  userId: string;
};

type User = {
  id: string;
  email: string;
  name: string;
};

type ApiError = {
  error: string;
  field?: string;
};
```

---

## Auth

### `POST /api/auth/register`

```
Body:
  {
    name: string       — requerido, min: 2 chars
    email: string      — requerido, formato email válido
    password: string   — requerido, min: 8 chars
  }

Response 201:
  { user: User }
  + Set-Cookie: session=... (httpOnly)

Response 400:
  { error: "Email already in use" }

Response 422:
  { error: "Validation failed", field: "email" | "password" | "name" }
```

### `POST /api/auth/login`

```
Body:
  {
    email: string,
    password: string
  }

Response 200:
  { user: User }
  + Set-Cookie: session=... (httpOnly)

Response 401:
  { error: "Invalid credentials" }
```

### `POST /api/auth/logout`

```
Response 200:
  { success: true }
  + Set-Cookie: session=; expires=Thu, 01 Jan 1970 (limpia cookie)
```

### `GET /api/auth/me`

```
— Verifica sesión activa y devuelve el usuario

Response 200: User

Response 401:
  { error: "Unauthorized" }

— Usado al cargar la app para saber si hay sesión activa
```

---

## Tasks

### `GET /api/tasks`

```
— Devuelve TODAS las tareas del usuario autenticado
— El filtrado (pendientes/completadas) se hace en el cliente

Response 200:
  {
    data: Task[]
  }

Response 401: { error: "Unauthorized" }

Nota: sin paginación en v1 — listas personales son cortas.
      Si superan los 200 items en el futuro, agregar cursor.
```

### `POST /api/tasks`

```
Body:
  {
    title: string           — requerido, min: 1 char, max: 280 chars
    description?: string    — opcional, max: 1000 chars
  }

Response 201: Task

Response 400:
  { error: "Validation failed", field: "title" }

Response 401: { error: "Unauthorized" }
```

### `PATCH /api/tasks/:id`

```
— Actualiza título, descripción o estado completed
— Solo enviar los campos que cambian

Body:
  {
    title?: string
    description?: string | null
    completed?: boolean
  }

Response 200: Task

Response 404: { error: "Not found" }

Response 403: { error: "Forbidden" }
— La tarea existe pero pertenece a otro usuario

Response 401: { error: "Unauthorized" }
```

### `DELETE /api/tasks/:id`

```
Response 204: (sin body)

Response 404: { error: "Not found" }

Response 403: { error: "Forbidden" }

Response 401: { error: "Unauthorized" }
```

---

## Implementación en el cliente

### Hooks de TanStack Query

```typescript
// hooks/queries/useTasks.ts
export function useTasksQuery() {
  return useQuery({
    queryKey: ["tasks"],
    queryFn: () => taskService.getAll(),
    staleTime: 30_000, // 30 segundos
  });
}

// hooks/queries/useCreateTask.ts
export function useCreateTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: taskService.create,
    onMutate: async (newTask) => {
      await queryClient.cancelQueries({ queryKey: ["tasks"] });
      const previous = queryClient.getQueryData(["tasks"]);

      // Optimistic update: agregar tarea inmediatamente
      queryClient.setQueryData(["tasks"], (old: { data: Task[] }) => ({
        data: [
          ...old.data,
          {
            id: crypto.randomUUID(), // ID temporal
            completed: false,
            createdAt: new Date().toISOString(),
            ...newTask,
          },
        ],
      }));

      return { previous };
    },
    onError: (err, newTask, context) => {
      // Rollback si falla
      queryClient.setQueryData(["tasks"], context?.previous);
    },
    onSettled: () => {
      // Sincronizar con el servidor siempre
      queryClient.invalidateQueries({ queryKey: ["tasks"] });
    },
  });
}

// hooks/queries/useToggleTask.ts
export function useToggleTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: ({ id, completed }: { id: string; completed: boolean }) =>
      taskService.update(id, { completed }),
    onMutate: async ({ id, completed }) => {
      await queryClient.cancelQueries({ queryKey: ["tasks"] });
      const previous = queryClient.getQueryData(["tasks"]);

      // Optimistic update: cambiar completed inmediatamente
      queryClient.setQueryData(["tasks"], (old: { data: Task[] }) => ({
        data: old.data.map((task) =>
          task.id === id ? { ...task, completed } : task,
        ),
      }));

      return { previous };
    },
    onError: (err, variables, context) => {
      queryClient.setQueryData(["tasks"], context?.previous);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ["tasks"] });
    },
  });
}

// hooks/queries/useDeleteTask.ts
export function useDeleteTask() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: taskService.delete,
    // Sin optimistic update — acción irreversible
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["tasks"] });
    },
  });
}
```

### Filtrado en el cliente

```typescript
// El filtro vive en Zustand, el filtrado se hace aquí
// No se hace una request nueva al servidor por cada filtro

function useFilteredTasks() {
  const { data } = useTasksQuery();
  const activeFilter = useAuthStore((state) => state.activeFilter);

  return useMemo(() => {
    if (!data?.data) return [];

    switch (activeFilter) {
      case "pending":
        return data.data.filter((t) => !t.completed);
      case "completed":
        return data.data.filter((t) => t.completed);
      default:
        return data.data;
    }
  }, [data, activeFilter]);
}
```

---

## Changelog

| Fecha      | Versión | Cambio           |
| ---------- | ------- | ---------------- |
| 2026-05-12 | v1.0    | Contrato inicial |
