# 03 — Actores y Roles

## Actores del sistema

| Actor | Descripción | Cómo interactúa |
|---|---|---|
| **Operador** | La persona que atiende el mostrador. Es también el dueño del local. | Usa la PC del local con teclado; carga pedidos, imprime, consulta el historial, edita el menú y hace copias de seguridad. |
| **Cliente final** (externo) | Quien hace el pedido. | No usa el sistema. Pide por WhatsApp; el operador carga el pedido. |
| **Sistema** (actor técnico) | Procesos automáticos. | Reserva el número de comanda, imprime, hace la copia de seguridad, calcula el día operativo. |

## RBAC — Matriz de permisos

No hay autenticación ni autorización. Un único usuario implícito con acceso total.

| Recurso | Operador | Notas |
|---|---|---|
| Menú (productos, presentaciones, códigos, precios) | CRUD (baja lógica) | El borrado físico no existe. |
| Pedidos | Crear, leer, anular, reimprimir | Nunca se eliminan. |
| Correcciones de pedido | Crear (segunda iteración) | Con snapshot y motivo. |
| Clientes | Crear, leer, actualizar | Se autocompletan por teléfono. |
| Configuración | Leer, actualizar | Impresora, ancho, letra, carpeta de backup, hora de corte. |
| Copia de seguridad | Ejecutar manualmente | Automática al cerrar y al abrir el día operativo. |

## Rutas públicas

No aplica: app de escritorio sin red ni superficie HTTP. Todas las pantallas son locales y accesibles sin login por diseño (una sola PC en un local).

## Implicancias de seguridad

- No hay secretos ni credenciales que administrar.
- El riesgo real no es el acceso: es la **pérdida o corrupción del archivo SQLite** en una PC que también se usa para WhatsApp. Se mitiga con copias automáticas y con la carpeta de backup fuera de la carpeta de la app.
- Si en el futuro hubiera más de una persona cargando pedidos, el modelo ya puede registrar autoría, pero no se implementa ahora.
