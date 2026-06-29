# DEMO · Detalle de Especies Municipales Online

Beta funcional de un SaaS para la gestión del inventario de **especies municipales** de una
alcaldía. En El Salvador, una *especie municipal* es un **documento administrativo oficial**
(vialidades, recibos de ingreso, avisos de cobro, solvencias, permisos, carnés, boletas,
licencias, constancias…), gestionado por correlativo, existencia y precio unitario.
Pensado como base escalable multi‑distrito / multi‑alcaldía.

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

## Despliegue en GitHub Pages

Una app de Vite **no se sirve desde el código fuente** (hay que compilarla). El repo incluye
un workflow que compila y publica `dist/` automáticamente. Pasos (una sola vez):

1. Sube el proyecto a GitHub (rama `main`).
2. En el repo: **Settings → Pages → Build and deployment → Source → "GitHub Actions"**.
3. Cada `git push` a `main` ejecuta el workflow `.github/workflows/deploy.yml` y publica el sitio.
4. La URL queda como `https://TU_USUARIO.github.io/TU_REPO/`.

Notas:
- `vite.config.ts` usa `base: './'` (rutas relativas) → funciona en cualquier subruta sin
  configurar el nombre del repo.
- Se usa `HashRouter`, así que las rutas viven en el hash (`…/#/dashboard`) y **funcionan al
  recargar** sin necesidad de reglas de servidor.
- Si antes pusiste *Source: Deploy from a branch*, cámbialo a **GitHub Actions** (esa opción
  servía el código fuente sin compilar → página en blanco).

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

- **Modo oscuro:** fondo negro puro `#000000`, sidebar *glassmorphism* (más ancho, con glow sutil).
- **Modo claro:** sin blancos puros (papel cálido `#F6F6F4`), apariencia premium.
- Acento **baby blue** en botones/enlaces (texto navy sobre relleno). El **rojo** se reserva
  para señalar stock bajo. Estética Linear / Vercel / Notion / Apple.
- Logotipo **DEMO** en tipografía manuscrita (Caveat). UI en Inter.
- El tema se persiste por dispositivo (`localStorage`) y respeta `prefers-color-scheme`
  y `prefers-reduced-motion`.

## Decisiones de UX destacadas

- **Dashboard centrado en la tabla** (~75%) + 3 módulos resumen discretos (~25%, sin iconos
  ni colores fuertes): En uso · Stock bajo · Inactivas. Debajo, "Últimas Entregas".
- Tabla principal: `#`, Especie, Código, Existencia, Precio Unitario, Valor Total, Último Corte
  (sin columna de estado). **Stock bajo** se comunica con la propia fila: texto rojo + borde
  rojo discreto, sin etiquetas.
- **Operaciones por modal** (Compra, Entrega, Modificar, Nueva especie): no se cambia de
  pantalla. Los ítems del sidebar abren el modal sobre la vista actual.
- El catálogo agrega documentos con existencia cero (sin pedir cantidades ni correlativos);
  el correlativo y los de entrega (`ENT-######`) se generan automáticamente.
