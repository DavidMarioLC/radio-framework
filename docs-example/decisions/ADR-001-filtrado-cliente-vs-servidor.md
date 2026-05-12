# ADR-001: Filtrado de tareas en cliente vs servidor

**Fecha:** 2026-05-12
**Estado:** Aprobado
**Área:** Networking / Data access

---

## Contexto

La pantalla principal permite filtrar tareas por estado:
todas, pendientes y completadas. Necesito decidir si el
filtrado se hace enviando un parámetro al servidor en cada
cambio de filtro, o si se carga todo una vez y se filtra
localmente en el cliente.

---

## Opciones consideradas

### Opción A: Filtrado en el servidor

Cada vez que el usuario cambia el filtro, se hace una nueva
request: `GET /api/tasks?status=pending`.

**A favor:** el servidor devuelve solo lo necesario, escala bien con miles de tareas, menos datos en memoria del cliente.

**En contra:** una request por cada cambio de filtro, latencia perceptible al cambiar entre Todas / Pendientes / Completadas, necesita loading state en cada cambio.

---

### Opción B: Filtrado en el cliente

Se carga la lista completa una sola vez con `GET /api/tasks`.
El filtrado se hace en el cliente con `.filter()` sobre los
datos ya en caché de TanStack Query.

**A favor:** cambio de filtro instantáneo sin latencia, una sola request al entrar a la pantalla, sin loading state al filtrar, más simple de implementar.

**En contra:** carga todos los datos aunque no se usen todos, no escala si el usuario tiene miles de tareas.

---

## Decisión

**Elegimos: Opción B — filtrado en el cliente**

Las tareas son datos personales de un solo usuario. Es realista
asumir que no tendrá más de 200-300 tareas en total. Filtrar
localmente es instantáneo y evita una round-trip al servidor
en cada interacción.

Si en el futuro detectamos usuarios con cientos de tareas y
performance degradada, podemos migrar a filtrado en servidor
agregando `?status=` al endpoint — el contrato ya lo contempla
como extensión futura.

---

## Consecuencias

**Positivas:**

- Cambiar entre filtros es instantáneo (0ms de latencia percibida)
- Solo una request al entrar a `/tasks`
- El código de filtrado es un `.filter()` con `useMemo` — simple y predecible

**Negativas / Trade-offs aceptados:**

- Si un usuario acumula 500+ tareas, el payload inicial crece
- Los datos en caché incluyen tareas que el filtro activo no muestra

**Tareas que genera esta decisión:**

- [ ] Implementar `useFilteredTasks` hook con `useMemo`
- [ ] `useAuthStore` guarda solo `activeFilter`, nunca las tareas filtradas
- [ ] Documentar el límite en `api-contracts.md`: sin paginación hasta 200 items

---

## Referencias

- `api-contracts.md` → sección GET /api/tasks (nota sobre paginación futura)
- ADR-002: Optimistic updates para crear y toggle tarea
