# 07 — Flujos Principales

## Flujo 1: Cargar y confirmar un pedido con impresión

**Disparador**: el operador abre la pantalla de pedido.
**Actor**: Operador.

**Pasos**:
1. La vista enfoca el campo código.
2. El operador escribe un código + Enter. `Core` busca la presentación activa por código.
3. `Core` arma un `PedidoItem` con código, nombre y precio congelados.
4. El ViewModel recalcula el total y lo muestra.
5. El operador repite 2–4, carga datos del cliente y forma de pago.
6. El operador confirma (Enter o atajo).
7. `Core` valida reglas (total > 0, dirección si es domicilio, forma de pago).
8. `Data` abre una transacción `BEGIN IMMEDIATE`, reserva el número del día operativo (`contador_comandas`), inserta pedido e ítems, y hace commit.
9. `Printing` construye `ComandaDocument` desde los datos guardados.
10. `ComandaLayoutEngine` calcula el layout con ancho y letra configurados.
11. `IComandaPrinter` (driver por defecto) imprime sin diálogo.
12. Se registra el resultado en `pedido_impresiones` y se muestra confirmación.

**Casos de error**:
| Caso | Manejo |
|---|---|
| Código inexistente o inactivo | Error en pantalla, no se agrega el ítem. |
| Total 0 al confirmar | Se bloquea la confirmación. |
| Domicilio sin dirección | Se marca el campo y se bloquea. |
| Sin impresora configurada | Aviso previo; se permite guardar sin imprimir. |
| La impresión falla | El pedido queda guardado; alerta con "Reimprimir"; se registra el error. |
| Corte de luz entre el commit y la impresión | El pedido existe; al abrir se puede reimprimir desde el historial. |

## Flujo 2: Reimprimir una comanda

**Disparador**: el operador busca un pedido y toca "Reimprimir".
**Actor**: Operador.

**Pasos**:
1. `Data` carga el pedido y sus ítems congelados.
2. `Core` valida que el pedido exista; si está anulado, lo marca como anulado en la salida.
3. `Printing` reconstruye el `ComandaDocument` desde los snapshots (no desde el menú actual).
4. Se imprime y se registra con motivo `reimpresion`.

**Casos de error**: impresora no disponible → se muestra el error y se permite reintentar; el registro queda con `exito = 0`.

## Flujo 3: Anular un pedido

**Disparador**: el operador elige "Anular" en el detalle de un pedido.
**Actor**: Operador.

**Pasos**:
1. Se pide confirmación y motivo opcional.
2. `Core` valida que el pedido no esté ya anulado.
3. `Data` actualiza `estado = 'anulado'`, `anulado_at`, `anulado_motivo`.
4. El historial refleja el estado sin eliminar la fila.

**Casos de error**: pedido ya anulado → se informa y no se modifica. Nunca se borra.

## Flujo 4: Editar el menú

**Disparador**: el dueño entra a "Menú".
**Actor**: Operador/dueño.

**Pasos**:
1. Se listan productos con sus presentaciones agrupadas.
2. Alta/edición: nombre, categoría, presentación, código, precio.
3. `Core` valida: 4 dígitos, único, precio ≥ 0.
4. `Data` guarda; si se desactiva, `activo = 0`.
5. Los pedidos existentes no se tocan (precio congelado).

**Casos de error**: código repetido (incluida una presentación inactiva) → se muestra qué producto lo usa; precio inválido → se rechaza.

## Flujo 5: Copia de seguridad

**Disparadores**: cierre de la app · primera apertura del día operativo · botón manual.
**Actor**: Sistema / Operador.

**Pasos**:
1. Se calcula la fecha operativa actual.
2. En apertura: si `app.ultima_copia_apertura_fecha` ≠ fecha operativa, se copia y se actualiza la marca.
3. En cierre o manual: se copia siempre.
4. La copia usa el mecanismo de backup de SQLite o copia el archivo con la base cerrada.
5. Se escribe en `backup_log` con motivo, ruta, éxito y tamaño.
6. Si la copia fue exitosa, se eliminan las copias con más de 30 días.

**Casos de error**: carpeta sin permisos o disco lleno → se registra el fallo, se avisa al operador y **no** se borran copias anteriores. El cierre no se bloquea.

## Flujo 6: Configurar impresora e imprimir prueba

**Disparador**: el operador entra a "Configuración → Impresión".
**Actor**: Operador.

**Pasos**:
1. Se listan las colas instaladas en Windows.
2. El operador elige una, el ancho de papel y el tamaño de letra.
3. "Imprimir prueba" construye una comanda de ejemplo y la envía por el camino seleccionado (`driver` o `escpos`).
4. Se registra el intento con motivo `prueba` y se informa el resultado.
5. Al guardar, la configuración queda activa para los próximos pedidos.

**Casos de error**: ninguna impresora instalada / spooler detenido → mensaje claro con instrucción; impresión fallida → se muestra el error de Windows y se sugiere probar el otro camino.
