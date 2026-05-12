# ADR-003: Rendering strategy — CSR para toda la app

**Fecha:** 2026-05-12
**Estado:** Aprobado
**Área:** Rendering

---

## Contexto

Next.js ofrece múltiples estrategias de rendering: CSR, SSR,
SSG e ISR. Necesito decidir cuál aplicar a cada página de
la app (login, register, tasks). La elección afecta el tiempo
de carga inicial, el SEO y la complejidad de implementación.

---

## Opciones consideradas

### Opción A: CSR para toda la app

Todas las páginas se renderizan en el cliente. El servidor
entrega un HTML mínimo y React toma el control desde el inicio.
Los datos se fetchen desde el cliente via TanStack Query.

**A favor:** simple de implementar sin lógica de servidor en páginas, los datos del usuario nunca pasan por el servidor de Next.js, modelo mental consistente en toda la app, suficiente para una app privada detrás de auth.

**En contra:** HTML inicial vacío — no apto para SEO, primera carga puede sentirse más lenta en conexiones lentas.

---

### Opción B: SSR para todas las páginas

Cada página se renderiza en el servidor en cada request.
Los datos se fetchen en el servidor y se inyectan en el
HTML inicial.

**A favor:** HTML completo desde el primer request, mejor TTFB para contenido crítico, útil si hubiera SEO.

**En contra:** más complejo — lógica de fetch duplicada en servidor y cliente, las páginas de auth no necesitan SSR, overhead innecesario para un dashboard privado.

---

### Opción C: SSR solo para /tasks, CSR para login y register

La pantalla principal de tareas usa SSR para cargar más rápido.
Las páginas de auth usan CSR porque son formularios simples.

**A favor:** optimiza la página más importante, login y register siguen siendo simples.

**En contra:** dos modelos de rendering en la misma app sin beneficio claro, `/tasks` es privada por lo que SSR no mejora SEO, mayor complejidad para gestionar sesión en servidor.

---

## Decisión

**Elegimos: Opción A — CSR para toda la app**

La app es completamente privada detrás de autenticación.
Ninguna página necesita SEO ni tiene contenido público que
justifique SSR. CSR es suficiente, más simple de mantener
y evita la complejidad de manejar sesión y datos del usuario
en el servidor de Next.js.

Next.js se usa en este proyecto por su sistema de routing,
API routes integradas y facilidad de deployment — no por
sus capacidades de server rendering.

---

## Consecuencias

**Positivas:**

- Todas las páginas siguen el mismo modelo — sin casos especiales
- Los datos del usuario se fetchen siempre desde el cliente con TanStack Query
- La lógica de autenticación vive solo en el middleware y en el cliente

**Negativas / Trade-offs aceptados:**

- El HTML inicial es un shell vacío — aceptable porque no hay SEO
- La primera carga puede sentirse más lenta en conexiones muy lentas,
  mitigado con skeleton screens en `/tasks`

**Tareas que genera esta decisión:**

- [ ] Configurar middleware de Next.js para proteger rutas privadas
- [ ] Implementar skeleton screens en `TaskList` para la carga inicial
- [ ] Asegurarse de que ninguna página use `getServerSideProps`

---

## Referencias

- [Next.js rendering strategies](https://nextjs.org/docs/pages/building-your-application/rendering)
- `architecture.md` → sección A, Decisión de rendering strategy
- ADR-001: Filtrado en cliente (consistente con el modelo CSR)
