# CandlePaw — API

Backend compartido (Express + PostgreSQL/MySQL) para el proyecto CandlePaw: una
tienda de velas artesanales con blog/comunidad, más un panel interno de
inventario y recetario. Reemplaza a los prototipos [`Candle-paw`](https://github.com/JoseMunozO/Candle-paw)
(archivado) y [`CandlePaw-DB`](https://github.com/JoseMunozO/CandlePaw-DB)
(archivado), que cubrían la misma idea por separado con tecnologías básicas.

## El producto, en tres piezas

```text
Cliente (navegador)                    Staff / admin (navegador)
      |                                        |
      v                                        v
candlepaw-store (React)              candlepaw-admin (React)
Tienda + blog + comentarios          Contenido (moderación) +
                                      Inventario + Recetario
      |                                        |
      +------------------+  +------------------+
                         |  |
                         v  v
                  candlepaw-api (este repo)
                  Express + PostgreSQL/MySQL
```

- [`candlepaw-store`](https://github.com/JoseMunozO/candlepaw-store) — cara
  pública: catálogo de velas, blog, comentarios de clientes, cuenta de
  usuario.
- [`candlepaw-admin`](https://github.com/JoseMunozO/candlepaw-admin) — panel
  interno con dos secciones: **Contenido** (qué posts/comentarios/productos
  se ven en la tienda) e **Inventario/Recetario** (stock de materiales y
  cálculo de costo/precio de cada vela según su receta).
- **candlepaw-api** (este repo) — una sola API para las dos apps, con
  autenticación por rol (cliente vs. staff/admin).

## Por qué una sola API para dos frontends

Cliente y admin comparten el mismo dato de fondo (productos, posts,
comentarios) pero con permisos distintos: un cliente ve solo contenido
publicado, un admin ve y modera todo. Separar los frontends pero compartir la
API evita duplicar lógica de negocio (sobre todo el cálculo de costos del
recetario) en dos sitios.

## Estado

Fase de planificación — ver [`docs/roadmap.md`](docs/roadmap.md) para el
modelo de datos, los endpoints y las fases de construcción. Todavía no hay
código.
