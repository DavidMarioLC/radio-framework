# /docs — Arquitectura de [Nombre del Proyecto]

> Este folder documenta las decisiones de diseño del sistema frontend.
> No es documentación de código — es documentación de **por qué**
> las cosas están como están.

---

## Estructura

```
/docs
  README.md                  ← estás aquí
  architecture.md            ← diseño completo con framework RADIO
  api-contracts.md           ← contratos entre frontend y servidor
  decisions/
    ADR-000-template.md      ← plantilla para nuevas decisiones
    ADR-001-[titulo].md      ← primera decisión arquitectural
    ADR-002-[titulo].md      ← segunda decisión arquitectural
```

---

## Cómo usar este sistema

**Al iniciar el proyecto:**
rellena `architecture.md` con al menos R, A y D antes de codear.
Deja I y O en borrador — se completan mientras se desarrolla.
La primera decisión no obvia que tomes → crea un ADR.

**Durante el desarrollo:**
si cambiaste de opinión sobre algo → actualiza `architecture.md`
y marca el ADR anterior como `Reemplazado por ADR-NNN`.
Si el contrato con el servidor cambia → actualiza `api-contracts.md`.

**Al terminar una feature:**
completa la sección O de `architecture.md` con las optimizaciones
que realmente aplicaste.

---

## Cuándo crear un ADR

Crea un ADR cuando la respuesta a _"¿por qué está hecho así?"_
no sea obvia leyendo el código.

**✅ Sí merece un ADR:**

- Elegir entre dos estrategias de state management
- Decidir entre filtrar en cliente o en servidor
- Elegir entre WebSockets, SSE o polling
- Decidir qué mutaciones usan optimistic updates
- Elegir la rendering strategy (CSR, SSR, SSG, ISR)
- Cambiar una decisión anterior

**❌ No merece un ADR:**

- Qué librería de componentes UI usar
- Convenciones de nombrado de archivos
- Configuración de ESLint, Prettier o Husky

---

## Numeración de ADRs

Los ADRs se numeran secuencialmente y **nunca se borran**.
Si una decisión cambia, el ADR original se marca como
`Reemplazado por ADR-NNN` y se crea uno nuevo.
Esto permite entender la evolución del proyecto,
no solo su estado actual.
