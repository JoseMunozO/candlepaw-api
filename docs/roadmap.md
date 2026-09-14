# Roadmap — CandlePaw API

API única que sirve a `candlepaw-store` (clientes) y `candlepaw-admin`
(staff). Reemplaza la lógica dispersa en los prototipos archivados
`Candle-paw` (sin backend) y `CandlePaw-DB` (Express + SQLite, solo
inventario).

## Stack

- Node.js + Express
- PostgreSQL o MySQL (a definir — MySQL si se prioriza consistencia con los
  otros proyectos del portfolio que ya usan MySQL; Postgres si se prioriza
  tipos de datos más ricos, p.ej. `numeric` exacto para costos)
- Auth: JWT + bcrypt (mismo patrón que `backend-preschool`)
- Migraciones versionadas (Flyway o equivalente para el motor elegido) —
  nada de `sync()`/auto-generación de esquema en producción

## Modelo de datos (borrador)

### Usuarios y roles
- `users`: id, email, password_hash, display_name, role (`customer` |
  `staff` | `admin`), created_at
  - Un único modelo de usuario con rol, no tablas separadas — simplifica
    login y matchea el patrón de `backend-preschool` (roles con rango).

### Tienda / catálogo
- `candles` (producto): id, name, slug, description, images, recipe_id (FK),
  status (`draft` | `visible` | `hidden`), margin_percent, final_price
  (calculado o sobreescrito a mano), created_at
- `blog_posts`: id, title, slug, body, author_id (FK users), candle_id (FK
  opcional, post ligado a un producto), status (`draft` | `published` |
  `hidden`), published_at
- `comments`: id, author_id (FK users, solo customers), post_id (FK
  opcional), candle_id (FK opcional), body, status (`pending` | `visible` |
  `hidden`), created_at
  - Un comentario cuelga de un post o de una vela, no de ambos.
  - `status` es la base de la moderación que pide el admin.

### Inventario y recetario
- `materials` (insumos/materiales): id, name, unit (`g` | `ml` | `unit`),
  cost_per_unit, stock_quantity, minimum_quantity, category, created_at
- `stock_movements`: id, material_id (FK), type (`in` | `out` |
  `adjustment`), quantity, reason, performed_by (FK users), created_at
  - Auditoría de cada cambio de stock — "gestionar el almacén de forma
    segura" significa trazabilidad, no solo un número de stock editable.
- `recipes`: id, candle_id (FK, 1:1 con la vela), name, notes
- `recipe_items`: recipe_id (FK), material_id (FK), quantity_used
  - `cost(candle) = sum(recipe_item.quantity_used * material.cost_per_unit)`
    para todos los items de su receta — este cálculo vive en el backend
    (no duplicado en el frontend admin) y se recalcula cada vez que cambia
    el precio de un material o la receta.

## Endpoints (borrador, agrupados)

```text
POST   /auth/register            (customer)
POST   /auth/login
GET    /auth/me

GET    /candles                  (solo visibles, público)
GET    /candles/:slug
GET    /admin/candles            (todas, cualquier status)
POST   /admin/candles
PATCH  /admin/candles/:id        (incluye status, margin_percent)

GET    /blog                     (solo published, público)
GET    /blog/:slug
GET    /admin/blog
POST   /admin/blog
PATCH  /admin/blog/:id

POST   /comments                 (customer autenticado, status=pending)
GET    /admin/comments?status=pending
PATCH  /admin/comments/:id       (moderar: visible/hidden)

GET    /admin/materials
POST   /admin/materials
PATCH  /admin/materials/:id
POST   /admin/materials/:id/movements   (registra entrada/salida/ajuste)

GET    /admin/recipes/:candleId
PUT    /admin/recipes/:candleId         (reemplaza items de la receta)
GET    /admin/recipes/:candleId/cost    (costo calculado en vivo)
```

## Fases

### Fase 0 — Setup
- [ ] Elegir motor de base de datos (Postgres vs MySQL) y confirmarlo aquí
- [ ] `npm init`, Express, estructura de carpetas (controllers/routes/config,
      como `webshop-najs`, o capas más explícitas como `backend-preschool` —
      a decidir según tamaño del proyecto)
- [ ] Migraciones versionadas + script de seed con datos de prueba
- [ ] Docker Compose para DB local

### Fase 1 — Auth y usuarios
- [ ] Registro/login de clientes (JWT)
- [ ] Roles (`customer`/`staff`/`admin`) y middleware de autorización
- [ ] Alta manual de usuarios staff/admin (sin self-signup para estos roles)

### Fase 2 — Inventario y recetario
- [ ] CRUD de materiales
- [ ] Movimientos de stock con auditoría (quién, cuándo, por qué)
- [ ] Alertas de stock bajo (minimum_quantity)
- [ ] CRUD de recetas (items = material + cantidad)
- [ ] Endpoint de costo calculado + precio sugerido (costo × margen)

### Fase 3 — Catálogo y blog
- [ ] CRUD de velas (candles), con status draft/visible/hidden
- [ ] CRUD de posts de blog, con status draft/published/hidden
- [ ] Vinculación opcional post ↔ vela

### Fase 4 — Comentarios y moderación
- [ ] Clientes pueden comentar en posts y velas (status=pending)
- [ ] Admin lista comentarios pendientes y modera (visible/hidden)

### Fase 5 — Pulido
- [ ] Paginación y búsqueda en catálogo/blog
- [ ] Subida de imágenes (velas, posts) — definir almacenamiento (disco vs.
      S3-compatible)
- [ ] Tests de la lógica de costeo (la parte más sensible a bugs numéricos)

### Fuera de alcance por ahora
- Checkout / pagos reales — el catálogo y el precio existen, pero procesar
  pagos es una fase futura separada, a evaluar cuando el resto esté sólido.
