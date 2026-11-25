# 05 — Reglas de Negocio

Cada regla tiene un código `RN-{DOMINIO}-{NN}` para trazabilidad desde historias y tests.

## Dominio: Menú (RN-MEN)

- **RN-MEN-01**: Toda presentación pertenece a un producto padre. No existen presentaciones huérfanas.
- **RN-MEN-02**: El código de una presentación tiene exactamente 4 dígitos y es único en todo el sistema, incluidas las presentaciones inactivas.
- **RN-MEN-03**: Cada presentación tiene su propio precio. La surtida puede tener un precio distinto al de la docena común.
- **RN-MEN-04**: Desactivar un producto o una presentación es una baja lógica (`activo = 0`). Nunca se borra.
- **RN-MEN-05**: Un producto o presentación inactivo no puede agregarse a un pedido nuevo, pero sigue siendo legible en pedidos históricos.
- **RN-MEN-06**: El precio se expresa en centavos enteros y no puede ser negativo.
- **RN-MEN-07**: Las opciones por producto (tipo de pan, tipo de queso, sin ingrediente) se definen en el editor de menú, no en código. No tienen precio adicional en la beta.

## Dominio: Pedidos (RN-PED)

- **RN-PED-01**: El total del pedido es la suma de los subtotales de sus ítems. No hay costo de envío, descuentos ni recargos.
- **RN-PED-02**: El subtotal de un ítem es `precio_unitario_centavos × cantidad`.
- **RN-PED-03**: **Precio congelado**: al agregar un ítem se copian a `pedido_items` el código, el nombre y el precio vigentes. Cambios posteriores en el menú no alteran el pedido.
- **RN-PED-04**: Un pedido confirmado no puede quedar con total 0.
- **RN-PED-05**: El teléfono es opcional en mostrador y recomendado en domicilio. No bloquea la confirmación.
- **RN-PED-06**: Si el tipo es `domicilio`, la dirección es obligatoria. Si es `mostrador`, se ignora.
- **RN-PED-07**: El horario estimado es texto libre, precargado con "ahora + 40 minutos" y siempre editable.
- **RN-PED-08**: La forma de pago es obligatoria (`efectivo` o `transferencia`) y se imprime en la comanda.
- **RN-PED-09**: Al escribir un teléfono que ya existe en `clientes`, se autocompletan nombre y último domicilio. Ambos campos siguen siendo editables.
- **RN-PED-10**: Al confirmar un pedido con un teléfono nuevo, se crea el cliente. Con un teléfono existente, se actualizan nombre y último domicilio con lo usado en ese pedido.

## Dominio: Comandas y numeración (RN-COM)

- **RN-COM-01**: El número de comanda es correlativo por **día operativo** y reinicia a 1 en el primer pedido de cada día operativo.
- **RN-COM-02**: El número se reserva dentro de una transacción `BEGIN IMMEDIATE`. Nunca se repite ni se saltea por un pedido anulado.
- **RN-COM-03**: La comanda imprime: número, hora de entrada, nombre, teléfono, domicilio (si aplica), código + producto por ítem, precio por ítem, total y forma de pago.
- **RN-COM-04**: La impresión es silenciosa: nunca abre un diálogo por pedido. La impresora destino es la configurada, no la predeterminada del sistema salvo que el operador la elija como tal.
- **RN-COM-05**: Si la impresión falla, el pedido **se guarda igual** en estado confirmado y la pantalla muestra el error con la acción "Reimprimir".
- **RN-COM-06**: La reimpresión usa los datos congelados del pedido, no el menú actual.
- **RN-COM-07**: Toda impresión y reimpresión se registra en `pedido_impresiones` con motivo, camino, éxito y error.
- **RN-COM-08**: El ancho de papel y el tamaño de letra son configurables y se aplican al layout de la comanda.
- **RN-COM-09**: "Imprimir prueba" imprime una comanda de ejemplo con los datos y el formato configurados, y registra el intento con motivo `prueba`.
- **RN-COM-10**: La app nunca se comunica directamente con el hardware ni depende de un modelo de impresora. Usa el spooler de Windows.

## Dominio: Anulación y corrección (RN-ANU)

- **RN-ANU-01**: Anular un pedido no lo borra: cambia `estado` a `anulado` y registra `anulado_at` y `anulado_motivo`.
- **RN-ANU-02**: Un pedido anulado no se puede reimprimir como comanda válida; la reimpresión debe marcarse visualmente como "ANULADO".
- **RN-ANU-03**: Un pedido anulado no se puede reactivar. Si el cliente vuelve a pedir, se carga un pedido nuevo.
- **RN-ANU-04**: Corregir un pedido guarda un snapshot antes/después en `pedido_correcciones`, con motivo, y reimprime automáticamente. (Segunda iteración.)

## Dominio: Clientes (RN-CLI)

- **RN-CLI-01**: El teléfono se normaliza quitando espacios, guiones, paréntesis y el signo `+`. Si empieza con `54`, se descarta el prefijo de país; si contiene `9` de móvil argentino, se descarta junto con el `15` cuando corresponde. La normalización es determinista y está cubierta por tests.
- **RN-CLI-02**: `telefono_normalizado` es único. Dos pedidos con el mismo teléfono escrito de formas distintas resuelven al mismo cliente.
- **RN-CLI-03**: El último domicilio se actualiza solo cuando el pedido es de tipo domicilio. Un pedido de mostrador no borra el domicilio guardado.

## Dominio: Copia de seguridad (RN-SEG)

- **RN-SEG-01**: Se hace una copia automática **al cerrar la app**.
- **RN-SEG-02**: Se hace una copia automática **la primera vez que se abre la app en cada día operativo**, si todavía no existe una copia de apertura para esa fecha.
- **RN-SEG-03**: Existe un botón "Copiar ahora" que ejecuta una copia manual.
- **RN-SEG-04**: La carpeta destino es configurable; el default es `Documentos\Vargas Pizza\Copia de seguridad`.
- **RN-SEG-05**: Se conservan las copias de los últimos 30 días. Las más viejas se eliminan después de una copia exitosa.
- **RN-SEG-06**: Una copia fallida nunca borra copias anteriores ni interrumpe el cierre de la app; queda registrada en `backup_log`.
- **RN-SEG-07**: La copia se hace con el mecanismo de backup online de SQLite (o copiando el archivo con la base cerrada), nunca copiando un archivo abierto a mano.

## Dominio: Configuración y operación (RN-CON)

- **RN-CON-01**: La hora de corte del día operativo se configura en `operacion.hora_corte`, default `06:00`.
- **RN-CON-02**: Cambiar la hora de corte no renumera pedidos existentes; solo afecta los nuevos y el disparador de backup.
- **RN-CON-03**: El cambio de impresora debe validarse con "Imprimir prueba" antes de considerarse configurado.

## Dominio: Excepciones globales (RN-GLO)

- **RN-GLO-01**: El sistema funciona 100 % offline. Ninguna funcionalidad puede depender de red, nube, telemetría ni servicios externos.
- **RN-GLO-02**: Ningún pedido se elimina físicamente, en ningún camino de código.
- **RN-GLO-03**: La app debe seguir siendo usable si WhatsApp está abierto y la PC tiene poca memoria libre.
- **RN-GLO-04**: Los datos del cliente (nombre, teléfono, domicilio) nunca salen de la PC del local.
