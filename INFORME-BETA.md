# Informe de la Beta — DEMO

**Detalle de Especies Municipales Online** · Beta 0.1
Rol asumido: Arquitecto de Software Senior · UX/UI Senior · Full‑Stack Engineer.

---

## 1. Arquitectura utilizada

**Enfoque:** SPA por capas con separación estricta de responsabilidades, preparada para
escalar a un SaaS multi‑alcaldía sin reescribir la capa de presentación.

```
Presentación (pages + components)
        │  consume
Hooks tipados (useAuth, useInventory, useTheme, useModal)
        │  exponen
Estado de aplicación (Contexts + useReducer)
        │  opera sobre
Dominio (types) + Lógica pura (lib/inventory.ts)
        │  alimentado por
Fuente de datos (data/mockData.ts ← futura API REST/GraphQL)
```

- **Modelo de dominio explícito y serializable** (`src/types`): `Species`, `InventoryItem`,
  `Movement` (ledger inmutable), `District`, `User`. Las formas imitan las respuestas de una
  API real → cambiar de mock a backend es un cambio de *data source*, no de UI.
- **Estado del inventario centralizado** en un `useReducer` (`InventoryContext`) con acciones
  puras (`ADD_TO_INVENTORY`, `CREATE_SPECIES`, `PURCHASE`, `DELIVER`, `ADJUST`). Una sola
  fuente de verdad; todo movimiento queda registrado en un *ledger*.
- **Derivación de estatus única** (`lib/inventory.ts → deriveStatus`): "En uso / Stock bajo /
  Inactiva" se calcula en un solo lugar y se reutiliza en dashboard, tablas, filtros y reportes.
- **Multi‑distrito** transversal: el inventario y los movimientos se filtran por el distrito
  activo (`useDistrictInventory`), base directa para multi‑tenancy.
- **Modales globales orquestados** (`ModalContext` + `OperationModals`): cualquier superficie
  (sidebar, fila de tabla, botón) abre una operación sin cambiar de pantalla.

**Capa de UI:** sistema de design tokens en CSS variables (`index.css`) mapeados a Tailwind
(`bg`, `surface`, `fg`, `accent`, `danger`…). Un único set de tokens dirige ambos temas, lo
que garantiza coherencia y un cambio de marca trivial.

## 2. Mejoras implementadas (más allá de lo solicitado)

- **Jerarquía del sidebar por secciones** (General · Operaciones · Análisis · Configuración)
  manteniendo exactamente los ítems pedidos, pero con mejor legibilidad.
- **Acciones del sidebar = modales** sobre la vista actual (no navegación), cumpliendo
  "no cambiar de pantalla innecesariamente".
- **Tabla del dashboard ordenada por existencia ascendente**: el stock crítico salta primero.
- **Validación de negocio en operaciones**: no se puede entregar más de lo disponible; las
  cantidades deben ser positivas; el ajuste sólo registra movimiento si cambia la existencia.
- **Correlativo automático** (`DC‑001`, `DN‑002`…) al añadir desde catálogo.
- **Estados vacíos** diseñados (catálogo sin especies locales, inventario vacío, sin movimientos).
- **Accesibilidad**: roles ARIA en el modal, `aria-label` en botones de icono, focus visible,
  `:focus-visible`, manejo de `Escape`, bloqueo de scroll, foco inicial dentro del diálogo,
  soporte de `prefers-reduced-motion` y `prefers-color-scheme`.
- **Datos semilla para ambos distritos** → las dos cuentas demo lucen completas.
- **Consola limpia** (flags `future` de React Router activados).

## 3. Decisiones de diseño tomadas

| Decisión | Razón |
|----------|-------|
| Acento **esmeralda** | Coherente con "especies" (botánico), sobrio y premium en ambos temas. |
| **Inter + Spectral** | Inter para UI/datos (números tabulares); Spectral (serif) para títulos = toque elegante sin ruido. |
| Modo claro en **papel cálido** `#F6F6F4` | Cumple "sin blancos puros" y reduce fatiga visual. |
| **Glassmorphism** sólo en sidebar/overlays | Estilo premium contenido; las tarjetas se mantienen discretas (prioriza información sobre decoración). |
| Línea roja = **borde inset + tinte** en toda la fila | Lectura inmediata del stock bajo sin saturar de iconos. |
| Login **sin selector de distrito** | El distrito se deriva de las credenciales (requisito) → menos fricción y más seguro. |
| Animaciones 200–300 ms, sólo `opacity`/`transform` | Fluidez sin distracción y buen rendimiento. |

## 4. Posibles mejoras futuras

- **Backend real** (REST/GraphQL) + persistencia; sustituir `data/mockData.ts` y conectar el
  reducer a una API (los tipos ya están listos).
- **Autenticación real** (JWT/OAuth, hash de contraseña, refresh, RBAC por permisos finos).
- **Multi‑tenancy** por alcaldía (aislamiento de datos, branding por organización).
- **Reportes avanzados**: rangos de fecha, exportación PDF/Excel, gráficas con librería dedicada.
- **Búsqueda global** funcional (la barra del topbar ya está maquetada).
- **Paginación/virtualización** de tablas para inventarios grandes.
- **Tests** (Vitest + Testing Library) sobre el reducer y los flujos de operación; e2e con Playwright.
- **i18n** y **PWA/offline** para trabajo de campo.
- **Auditoría** ampliada (quién/cuándo/IP) sobre el ledger de movimientos ya existente.

## 5. Código preparado para futuras funcionalidades

- **Ledger de movimientos** (`Movement`): ya registra compras, entregas y ajustes con autor y
  fecha → base directa de auditoría e historial.
- **Roles** (`User.role: 'admin' | 'operator'`) y `districtIds`: cimiento para permisos y
  multi‑distrito (el selector ya se bloquea con un solo distrito).
- **`adjustInventory`** acepta `minStock` opcional → preparado para gestión de umbrales.
- **Tokens de tema** centralizados → theming/white‑label por alcaldía sin tocar componentes.
- **Primitivos UI desacoplados** (`components/ui`) → acelera nuevas pantallas con consistencia.
- **`ModalContext`** admite nuevas operaciones añadiendo un caso en `OperationModals`.

---

### Verificación realizada

- ✅ `tsc --noEmit` sin errores · `vite build` correcto (JS 243 KB / gzip 75 KB).
- ✅ Sin errores en consola del navegador.
- ✅ Flujo de compra validado end‑to‑end (existencia y ledger actualizados).
- ✅ Login, bloqueo de distrito (operador) y selector multi‑distrito (admin).
- ✅ Modo claro (papel cálido) y oscuro (negro puro) verificados por estilos computados.
- ✅ Responsive desktop (1440) y tablet (768, sidebar → drawer) verificados.
