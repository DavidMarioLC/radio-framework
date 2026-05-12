# ADR-002: Optimistic updates en crear y toggle, no en eliminar

**Fecha:** 2026-05-12
**Estado:** Aprobado
**Área:** Networking / UX

---

## Contexto

La app tiene 3 mutaciones principales: crear tarea, marcar
como completada/pendiente (toggle) y eliminar. Necesito
decidir cuáles usan optimistic updates — el cliente actualiza
la UI antes de que el servidor confirme — y cuáles esperan
la respuesta del servidor antes de actuar.

---

## Opciones consideradas

### Opción A: Optimistic updates en todas las mutaciones

Crear, toggle y eliminar actualizan la UI inmediatamente.
Si el servidor falla, se hace rollback automático.

**A favor:** todas las acciones se sienten instantáneas, experiencia más fluida en general.

**En contra:** eliminar con rollback es confuso — el usuario ve reaparecer una tarea que ya "borró", eliminar es irreversible y el error tiene mayor costo perceptible.

---

### Opción B: Optimistic updates solo en crear y toggle

Crear y toggle son instantáneos en la UI. Eliminar espera
confirmación del servidor antes de quitar el item de la lista.

**A favor:** crear y toggle se sienten inmediatos, eliminar es predecible sin rollbacks sorpresivos, la lógica de rollback solo existe en 2 hooks en lugar de 3.

**En contra:** eliminar tiene ~200-300ms de latencia visible, pequeña inconsistencia de velocidad entre las distintas acciones.

---

### Opción C: Sin optimistic updates en ninguna mutación

Todas las mutaciones muestran un spinner y esperan al servidor.

**A favor:** más simple de implementar, el estado siempre refleja la realidad del servidor.

**En contra:** marcar una tarea como completada tarda 300ms — inaceptable para una acción que se hace decenas de veces por día.

---

## Decisión

**Elegimos: Opción B — optimistic en crear y toggle, no en eliminar**

Crear y toggle son acciones frecuentes, reversibles en caso
de error, y donde la latencia se nota más. El optimistic
update mejora la UX sin riesgo real.

Eliminar es irreversible. Si el servidor falla y hacemos
rollback, el usuario ve reaparecer una tarea que "ya borró"
— eso genera confusión y desconfianza en la app. Preferimos
los 200-300ms de espera a cambio de certeza.

---

## Consecuencias

**Positivas:**

- Crear y toggle se sienten instantáneos
- El flujo de eliminar es predecible y sin sorpresas
- La lógica de rollback vive solo en `useCreateTask` y `useToggleTask`

**Negativas / Trade-offs aceptados:**

- Eliminar es marginalmente más lento que crear o toggle
- `TaskItem` necesita mostrar el botón eliminar en estado `isPending`
  para que el usuario sepa que algo está pasando mientras espera

**Tareas que genera esta decisión:**

- [ ] `useCreateTask` y `useToggleTask` implementan `onMutate` con rollback
- [ ] `useDeleteTask` solo implementa `onSuccess` + `invalidateQueries`
- [ ] `TaskItem` deshabilita el botón eliminar mientras `isPending` es true

---

## Referencias

- `api-contracts.md` → sección de implementación de optimistic updates
- ADR-001: Filtrado en cliente
- ADR-003: Rendering strategy
