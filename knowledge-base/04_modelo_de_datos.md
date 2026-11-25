# 04 — Modelo de Datos

## Dominios

| Dominio | Responsabilidad |
|---|---|
| **Menú** | Productos, presentaciones con código y precio, y opciones configurables por producto. |
| **Clientes** | Identificación por teléfono normalizado y último domicilio conocido. |
| **Pedidos** | Cabecera, ítems con precio congelado, estado y auditoría. |
| **Comandas** | Numeración por día operativo, registro de impresiones y reimpresiones. |
| **Configuración** | Impresora, ancho de papel, tamaño de letra, hora de corte, carpeta de backup. |
| **Mantenimiento** | Migraciones de esquema y bitácora de copias de seguridad. |

## Convenciones

- **Dinero**: `INTEGER` en centavos. Nunca `REAL`/float.
- **Códigos**: `TEXT` de exactamente 4 dígitos (se preservan ceros a la izquierda).
- **Fechas y horas**: `TEXT` ISO-8601 en hora local (`YYYY-MM-DDTHH:MM:SS`). Un solo local, una sola zona horaria; no se usa UTC para evitar conversiones en un sistema monousuario.
- **Fechas operativas**: `TEXT` `YYYY-MM-DD`, calculadas como `fecha_hora_local - hora_de_corte`.
- **Booleanos**: `INTEGER` 0/1 con `CHECK`.
- **Borrado**: no existe borrado físico de pedidos. Los productos y presentaciones se desactivan (`activo = 0`).

## ERD

```
productos 1──N presentaciones
    │
    └──1──N producto_opciones 1──N opcion_valores

clientes 1──N pedidos
                 │
                 ├──1──N pedido_items ──1──N pedido_item_opciones
                 ├──1──N pedido_correcciones
                 └──1──N pedido_impresiones

contador_comandas   (fecha_operativa PK, ultimo_numero)
configuracion       (clave PK, valor)
backup_log          (bitácora de copias)
schema_migrations   (versión de esquema)
```

## Entidades

### `productos`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT |
| `nombre` | TEXT | NOT NULL |
| `categoria` | TEXT | NULL (ej. "Empanadas", "Pizzas", "Lomos") |
| `activo` | INTEGER | NOT NULL DEFAULT 1, CHECK IN (0,1) |
| `orden` | INTEGER | NOT NULL DEFAULT 0 |
| `created_at`, `updated_at` | TEXT | NOT NULL |

Índices: `(activo, orden)`. Relación: 1–N con `presentaciones` y `producto_opciones`.

### `presentaciones`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK AUTOINCREMENT |
| `producto_id` | INTEGER | NOT NULL, FK `productos(id)` |
| `nombre` | TEXT | NOT NULL (ej. "Unidad", "Media docena", "Docena", "Surtida") |
| `codigo` | TEXT | NOT NULL, `length(codigo) = 4`, solo dígitos, **UNIQUE global** |
| `precio_centavos` | INTEGER | NOT NULL, CHECK >= 0 |
| `activo` | INTEGER | NOT NULL DEFAULT 1, CHECK IN (0,1) |
| `orden` | INTEGER | NOT NULL DEFAULT 0 |
| `created_at`, `updated_at` | TEXT | NOT NULL |

Índices: `UNIQUE(codigo)`, `(producto_id, activo, orden)`.
**Por qué el código es único global**: evita ambigüedad al tipear. Si un código pertenece a una presentación inactiva, el editor de menú debe avisar antes de reutilizarlo.

### `producto_opciones` (segunda iteración)
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `producto_id` | INTEGER | NOT NULL, FK `productos(id)` |
| `nombre` | TEXT | NOT NULL (ej. "Tipo de pan", "Tipo de queso", "Sin ingrediente") |
| `tipo` | TEXT | NOT NULL, CHECK IN ('single','multi') |
| `requerido` | INTEGER | NOT NULL DEFAULT 0, CHECK IN (0,1) |
| `activo` | INTEGER | NOT NULL DEFAULT 1 |
| `orden` | INTEGER | NOT NULL DEFAULT 0 |

### `opcion_valores` (segunda iteración)
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `opcion_id` | INTEGER | NOT NULL, FK `producto_opciones(id)` |
| `nombre` | TEXT | NOT NULL (ej. "Árabe", "Francés", "Sin cebolla") |
| `activo` | INTEGER | NOT NULL DEFAULT 1 |
| `orden` | INTEGER | NOT NULL DEFAULT 0 |

**Sin precio adicional.** El cliente no confirmó que existan opciones con costo. Si lo confirma, se agrega `precio_extra_centavos` con una migración y se extiende el snapshot de `pedido_item_opciones`.

### `clientes`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `telefono_normalizado` | TEXT | NOT NULL, UNIQUE (solo dígitos) |
| `telefono_raw` | TEXT | NULL (como lo escribió el operador) |
| `nombre` | TEXT | NULL |
| `ultimo_domicilio` | TEXT | NULL |
| `created_at`, `updated_at` | TEXT | NOT NULL |

Índice: `UNIQUE(telefono_normalizado)`.
Normalización: se quitan espacios, guiones, paréntesis, `+`, y se descarta el prefijo de país cuando corresponde. El detalle exacto se define en RN-CLI-01.

### `pedidos`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `fecha_operativa` | TEXT | NOT NULL `YYYY-MM-DD` |
| `numero_dia` | INTEGER | NOT NULL, CHECK > 0 |
| `fecha_hora` | TEXT | NOT NULL (hora local real de carga) |
| `tipo` | TEXT | NOT NULL, CHECK IN ('mostrador','domicilio') |
| `cliente_id` | INTEGER | NULL, FK `clientes(id)` |
| `nombre` | TEXT | NULL |
| `telefono` | TEXT | NULL |
| `direccion` | TEXT | NULL (obligatoria si `tipo='domicilio'`, ver RN-PED-06) |
| `horario_estimado` | TEXT | NULL (texto libre, ej. "20:40") |
| `forma_pago` | TEXT | NOT NULL, CHECK IN ('efectivo','transferencia') |
| `estado` | TEXT | NOT NULL DEFAULT 'confirmado', CHECK IN ('confirmado','anulado') |
| `total_centavos` | INTEGER | NOT NULL, CHECK >= 0 |
| `anulado_at` | TEXT | NULL |
| `anulado_motivo` | TEXT | NULL |
| `created_at`, `updated_at` | TEXT | NOT NULL |

Índices: `UNIQUE(fecha_operativa, numero_dia)`, `(fecha_hora)`, `(estado)`, `(telefono)`.

### `pedido_items`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `pedido_id` | INTEGER | NOT NULL, FK `pedidos(id)` |
| `presentacion_id` | INTEGER | NULL, FK `presentaciones(id)` (null si se borró del menú) |
| `codigo_snapshot` | TEXT | NOT NULL |
| `nombre_snapshot` | TEXT | NOT NULL (producto + presentación, ej. "Empanadas — Docena") |
| `precio_unitario_centavos` | INTEGER | NOT NULL |
| `cantidad` | INTEGER | NOT NULL, CHECK > 0 |
| `subtotal_centavos` | INTEGER | NOT NULL, CHECK >= 0 |
| `nota` | TEXT | NULL (sale impresa) |
| `orden` | INTEGER | NOT NULL |

Índice: `(pedido_id, orden)`.

### `pedido_item_opciones`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `pedido_item_id` | INTEGER | NOT NULL, FK `pedido_items(id)` |
| `opcion_nombre` | TEXT | NOT NULL (snapshot) |
| `valor_nombre` | TEXT | NOT NULL (snapshot) |

### `pedido_correcciones` (segunda iteración)
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `pedido_id` | INTEGER | NOT NULL, FK `pedidos(id)` |
| `fecha_hora` | TEXT | NOT NULL |
| `motivo` | TEXT | NULL |
| `snapshot_antes` | TEXT | NOT NULL (JSON) |
| `snapshot_despues` | TEXT | NOT NULL (JSON) |
| `reimprimio` | INTEGER | NOT NULL DEFAULT 0 |

### `pedido_impresiones`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `pedido_id` | INTEGER | NULL, FK `pedidos(id)` (null en impresión de prueba) |
| `fecha_hora` | TEXT | NOT NULL |
| `motivo` | TEXT | NOT NULL, CHECK IN ('inicial','reimpresion','prueba') |
| `camino` | TEXT | NOT NULL, CHECK IN ('driver','escpos','file') |
| `exito` | INTEGER | NOT NULL |
| `error` | TEXT | NULL |

Índice: `(pedido_id, fecha_hora)`. Sirve para diagnosticar problemas de impresión en el local.

### `contador_comandas`
| Campo | Tipo | Constraints |
|---|---|---|
| `fecha_operativa` | TEXT | PK `YYYY-MM-DD` |
| `ultimo_numero` | INTEGER | NOT NULL DEFAULT 0 |

Se incrementa con `BEGIN IMMEDIATE` para que dos impresiones simultáneas no repitan número. Un pedido anulado **no libera** su número.

### `configuracion`
| Clave | Default | Descripción |
|---|---|---|
| `impresion.impresora_nombre` | `NULL` | Nombre exacto de la cola de Windows. |
| `impresion.modo` | `driver` | `driver` (principal) o `escpos` (respaldo). |
| `impresion.ancho_papel_mm` | `80` | Ancho físico del papel. |
| `impresion.tamano_letra_pt` | `9` | Tamaño de fuente de la comanda. |
| `operacion.hora_corte` | `06:00` | Hora de corte del día operativo. |
| `backup.carpeta` | `%USERPROFILE%\Documents\Vargas Pizza\Copia de seguridad` | Destino de las copias. |
| `backup.retencion_dias` | `30` | Días de copias que se conservan. |
| `app.ultima_copia_apertura_fecha` | `NULL` | Última fecha operativa en que se hizo la copia de apertura. |

### `backup_log`
| Campo | Tipo | Constraints |
|---|---|---|
| `id` | INTEGER | PK |
| `fecha_hora` | TEXT | NOT NULL |
| `motivo` | TEXT | NOT NULL, CHECK IN ('cierre','apertura','manual') |
| `ruta` | TEXT | NOT NULL |
| `exito` | INTEGER | NOT NULL |
| `error` | TEXT | NULL |
| `tamano_bytes` | INTEGER | NULL |

### `schema_migrations`
| Campo | Tipo | Constraints |
|---|---|---|
| `version` | INTEGER | PK |
| `applied_at` | TEXT | NOT NULL |

## Seed data inicial

Al crear la base por primera vez:

1. `schema_migrations` con `version = 1`.
2. `configuracion` con los defaults de la tabla anterior, excepto `impresion.impresora_nombre` (se elige en la pantalla de configuración).
3. **Ningún producto.** El menú real lo carga el dueño en el editor; depende de la lista de productos, códigos y precios que todavía no está digitalizada.

## Día operativo

```
fecha_operativa(fecha_hora_local, hora_corte) =
    date( fecha_hora_local - hora_corte )
```

Con corte 06:00, un pedido de las 01:30 del 26/11 pertenece al día operativo **25/11**. Se usa para:
- `pedidos.fecha_operativa` y `numero_dia`.
- `contador_comandas.fecha_operativa`.
- El disparador de la copia de seguridad diaria.
- El filtro por defecto del historial ("hoy" = día operativo actual).

Cambiar la hora de corte **no renumera** pedidos existentes: solo afecta los nuevos.
