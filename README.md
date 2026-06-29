# DEMO · Detalle de Especies Municipales Online

Beta funcional de un SaaS para la gestión del inventario de especies (viveros municipales)
de una alcaldía. Pensado como base escalable multi‑distrito / multi‑alcaldía.

> Estado: **Beta 0.1** — datos en memoria (no persistentes). Diseñada para crecer hacia el
> producto final sin reescribir la capa de presentación.

---

## Stack

| Capa | Tecnología |
|------|-----------|
| UI | React 18 + TypeScript |
| Build | Vite 5 |
| Estilos | Tailwind CSS 3 (tokens semánticos vía CSS variables) |
| Routing | React Router 6 |
| Estado | React Context + `useReducer` (sin dependencias externas) |

## Puesta en marcha

```bash
npm install
npm run dev      # http://localhost:5173
npm run build    # typecheck + build de producción
npm run preview  # sirve el build
```

## Acceso de demostración

El distrito se asigna **automáticamente** según las credenciales (no se elige en el login).
La contraseña es ignorada en la beta (cualquier valor no vacío sirve).

| Usuario | Rol | Distritos |
|---------|-----|-----------|
| `admin@demo.gob` | Admin | Multi‑distrito (selector activo) |
| `norte@demo.gob` | Operador | Distrito único (selector bloqueado) |

## Arquitectura (separación por capas)

```
src/
├── types/         Modelo de dominio (Species, InventoryItem, Movement, District, User)
├── data/          Datos semilla (mock) con la forma de la futura API
├── lib/           Lógica pura y utilidades (inventory, format, cn)
├── contexts/      Estado de aplicación (Theme, Auth, Inventory, Modal)
├── hooks/         Acceso tipado a los contexts (useAuth, useInventory, useTheme)
├── components/
│   ├── ui/        Primitivos reutilizables (Button, Card, Modal, Field, Badge…)
│   ├── layout/    Shell (Sidebar glass, Topbar, AppLayout, DistrictSelector)
│   ├── inventory/ Componentes de dominio (tabla, tarjetas, movimientos)
│   ├── catalog/   Tarjeta de especie
│   └── modals/    Operaciones (Compra, Entrega, Modificar, Nueva especie)
└── pages/         Pantallas (Login, Dashboard, Inventario, Catálogo, Reportes…)
```

**Principios aplicados:** SOLID · DRY · KISS · componentización · una sola fuente de verdad
para el estado del inventario (reducer) y para la derivación de estatus (`lib/inventory.ts`).

## Identidad visual

- **Modo oscuro:** fondo negro puro `#000000`, sidebar con *glassmorphism*, tarjetas discretas.
- **Modo claro:** sin blancos puros (papel cálido `#F6F6F4`), apariencia premium.
- Acento **esmeralda** (botánico, sobrio). Tipografía: Inter (UI) + Spectral (display).
- El tema se persiste por dispositivo (`localStorage`) y respeta `prefers-color-scheme`
  y `prefers-reduced-motion`.

## Decisiones de UX destacadas

- **Operaciones por modal** (Compra, Entrega, Modificar, Nueva especie): no se cambia de
  pantalla. Los ítems del sidebar abren el modal sobre la vista actual.
- **Stock bajo** resaltado con línea roja + tinte en toda la fila.
- El catálogo agrega especies con existencia cero (sin pedir cantidades ni correlativos);
  el correlativo se genera automáticamente.
