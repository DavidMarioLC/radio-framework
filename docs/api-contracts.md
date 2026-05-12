# API Contracts — [Nombre del Proyecto]

> El servidor se trata como caja negra.
> Este documento define qué le enviamos y qué esperamos recibir.
> Actualízalo cuando el contrato cambie, no después.

---

## Convenciones

- **Base URL:** `/api` o `/api/v1`
- **Autenticación:** [httpOnly cookie / Bearer token en header]
- **Paginación:** [cursor-based / offset]
- **Fechas:** ISO 8601 — `2026-05-12T10:30:00Z`
- **IDs:** [UUID v4 / CUID / autoincrement]
- **Errores:** `{ error: string, field?: string }`

---

## Tipos base

```typescript
// Copia esto en src/types/api.ts y ajusta según tu proyecto

type PaginatedResponse<T> = {
  data: T[];
  nextCursor: string | null;
  hasMore: boolean;
};

type ApiError = {
  error: string;
  field?: string; // para errores de validación de campo específico
  code?: string; // código de error interno (opcional)
};

type Timestamps = {
  createdAt: string; // ISO 8601
  updatedAt: string; // ISO 8601
};
```

---

## Entidades

```typescript
// Define aquí los tipos de las entidades principales

type[Entidad] =
  {
    id: string,
    // campos de la entidad
  } & Timestamps;
```

---

## Auth (si aplica)

### `POST /api/auth/register`

```
Body:
  {
    name: string,
    email: string,
    password: string
  }

Response 201: { user: User }
Response 400: { error: "Email already in use" }
Response 422: { error: "Validation failed", field: string }
```

### `POST /api/auth/login`

```
Body:
  {
    email: string,
    password: string
  }

Response 200: { user: User }
Response 401: { error: "Invalid credentials" }
```

### `POST /api/auth/logout`

```
Response 200: { success: true }
```

### `GET /api/auth/me`

```
Response 200: User
Response 401: { error: "Unauthorized" }
```

---

## [Recurso principal]

### `GET /api/[recurso]`

```
Query params:
  cursor?  : string   — paginación por cursor (omitir en primera carga)
  limit?   : number   — default: 20, max: 50
  [filtro]?: string   — filtros específicos del recurso

Response 200: PaginatedResponse<[Entidad]>
Response 401: { error: "Unauthorized" }
```

### `GET /api/[recurso]/:id`

```
Response 200: [Entidad]
Response 404: { error: "Not found" }
Response 401: { error: "Unauthorized" }
```

### `POST /api/[recurso]`

```
Body:
  {
    [campo]: [tipo],    — requerido
    [campo]?: [tipo]    — opcional
  }

Response 201: [Entidad]
Response 400: { error: "Validation failed", field: string }
Response 401: { error: "Unauthorized" }
```

### `PATCH /api/[recurso]/:id`

```
Body: Partial<Pick<[Entidad], '[campo]' | '[campo]'>>

Response 200: [Entidad]
Response 404: { error: "Not found" }
Response 403: { error: "Forbidden" }
Response 401: { error: "Unauthorized" }
```

### `DELETE /api/[recurso]/:id`

```
Response 204: (sin body)
Response 404: { error: "Not found" }
Response 403: { error: "Forbidden" }
Response 401: { error: "Unauthorized" }
```

---

## Comunicación en tiempo real (si aplica)

### WebSocket — `wss://[host]/ws`

```
// Eventos servidor → cliente
{ type: "ENTIDAD_CREATED", payload: [Entidad] }
{ type: "ENTIDAD_UPDATED", payload: Partial<[Entidad]> & { id: string } }
{ type: "ENTIDAD_DELETED", payload: { id: string } }

// Eventos cliente → servidor
{ type: "PING" }
```

### SSE — `GET /api/[recurso]/stream`

```
event: entidad_created
data: { "id": "...", ... }

event: entidad_updated
data: { "id": "...", ... }
```

> Elimina la sección que no uses.

---

## Patrones de implementación

### Query base con TanStack Query

```typescript
// hooks/queries/use[Entidad]Query.ts
export function use[Entidad]Query() {
  return useQuery({
    queryKey: ['[recurso]'],
    queryFn: () => [nombre]Service.getAll(),
    staleTime: [X],
  })
}
```

### Mutación con optimistic update

```typescript
// hooks/queries/useCreate[Entidad].ts
export function useCreate[Entidad]() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: [nombre]Service.create,
    onMutate: async (newItem) => {
      await queryClient.cancelQueries({ queryKey: ['[recurso]'] })
      const previous = queryClient.getQueryData(['[recurso]'])

      queryClient.setQueryData(['[recurso]'], (old) => ({
        // aplica el update optimista aquí
      }))

      return { previous }
    },
    onError: (err, newItem, context) => {
      queryClient.setQueryData(['[recurso]'], context?.previous)
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['[recurso]'] })
    },
  })
}
```

### Mutación sin optimistic update

```typescript
// Para acciones irreversibles (delete) o poco frecuentes
export function useDelete[Entidad]() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: [nombre]Service.delete,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['[recurso]'] })
    },
  })
}
```

---

## Changelog

| Fecha      | Versión | Cambio           |
| ---------- | ------- | ---------------- |
| YYYY-MM-DD | v1.0    | Contrato inicial |
