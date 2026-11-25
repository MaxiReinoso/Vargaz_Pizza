# 06 — Funcionalidades

Organizadas por épica y por historia de usuario (`US-NNN`). El **orden de las épicas es el orden de construcción acordado**: primero el flujo núcleo, después el resto de la beta.

## Épica 1 — Núcleo operable (beta, primera iteración)

### US-001 — Configurar la impresora e imprimir prueba
**Como** operador
**Quiero** elegir la impresora instalada en Windows y mandar una comanda de prueba
**Para** saber que la impresión silenciosa funciona antes de tomar el primer pedido

**Criterios de aceptación**:
- [ ] La pantalla lista todas las colas de impresión instaladas en Windows.
- [ ] Se puede seleccionar una y guardarla en `impresion.impresora_nombre`.
- [ ] "Imprimir prueba" imprime una comanda de ejemplo **sin abrir ningún diálogo**.
- [ ] Se configuran ancho de papel y tamaño de letra, y la prueba refleja ambos.
- [ ] Si la impresión falla, se muestra el error de Windows sin cerrar la pantalla.
- [ ] Se puede cambiar el modo `driver` ↔ `escpos`.

**Reglas**: RN-COM-04, RN-COM-08, RN-COM-09, RN-COM-10, RN-CON-03

### US-002 — Editar el menú
**Como** dueño
**Quiero** crear, editar y desactivar productos, presentaciones, códigos y precios
**Para** mantener la carta al día sin depender del desarrollador

**Criterios de aceptación**:
- [ ] Alta de producto con nombre y categoría opcional.
- [ ] Alta de presentaciones bajo un producto, cada una con código de 4 dígitos y precio.
- [ ] El sistema rechaza códigos repetidos, incompletos o no numéricos, e indica cuál presentación ya lo usa.
- [ ] Editar un precio no modifica pedidos existentes.
- [ ] Desactivar un producto o presentación no lo borra y no rompe el historial.
- [ ] Se ven agrupadas las presentaciones de un mismo producto.

**Reglas**: RN-MEN-01 a RN-MEN-06

### US-003 — Cargar un pedido por código
**Como** operador
**Quiero** escribir el código de 4 dígitos y que el ítem se agregue sin usar el mouse
**Para** cargar pedidos rápido en hora pico

**Criterios de aceptación**:
- [ ] El foco arranca en el campo código al abrir la pantalla de pedido.
- [ ] Escribir 4 dígitos + Enter agrega el ítem con cantidad 1.
- [ ] Un código inexistente o inactivo muestra un error claro y no agrega nada.
- [ ] Se puede cambiar la cantidad y agregar una nota libre por ítem.
- [ ] Se puede quitar un ítem antes de confirmar.
- [ ] Se cargan nombre, teléfono, tipo, dirección, horario estimado y forma de pago.
- [ ] Tab y Shift+Tab navegan entre campos; hay atajo para confirmar.
- [ ] La carga completa de un pedido de 4 ítems se hace sin mouse.

**Reglas**: RN-PED-05 a RN-PED-09, RN-MEN-05

### US-004 — Ver el total automático
**Como** operador
**Quiero** que el total se actualice al agregar, quitar o cambiar cantidades
**Para** no hacer cuentas a mano

**Criterios de aceptación**:
- [ ] El total se recalcula en < 150 ms desde cada cambio.
- [ ] El total es la suma de subtotales, sin envío ni recargos.
- [ ] Al confirmar, el total se guarda en `pedidos.total_centavos`.
- [ ] Un pedido con total 0 no se puede confirmar.

**Reglas**: RN-PED-01 a RN-PED-04

### US-005 — Confirmar e imprimir la comanda
**Como** operador
**Quiero** que al confirmar el pedido se imprima solo en la térmica
**Para** que la cocina la tenga sin que yo toque nada

**Criterios de aceptación**:
- [ ] Al confirmar se asigna el número del día operativo y se guarda el pedido.
- [ ] Se imprime automáticamente, sin diálogo, en la impresora configurada.
- [ ] La comanda incluye número, hora, nombre, teléfono, domicilio si aplica, código + producto, precio por ítem, total y forma de pago.
- [ ] Si la impresora falla, el pedido queda guardado y se ofrece "Reimprimir".
- [ ] Cada intento queda registrado en `pedido_impresiones`.
- [ ] Si no hay impresora configurada, se avisa antes de confirmar y se permite guardar sin imprimir.

**Reglas**: RN-COM-01 a RN-COM-07, RN-PED-03

### US-006 — Buscar cliente por teléfono
**Como** operador
**Quiero** que al escribir el teléfono aparezcan nombre y último domicilio
**Para** no preguntar los datos de nuevo

**Criterios de aceptación**:
- [ ] Al salir del campo teléfono, si existe un cliente se autocompletan nombre y último domicilio.
- [ ] Los campos autocompletados siguen siendo editables.
- [ ] Teléfonos escritos con espacios, guiones, `+54` o `15` resuelven al mismo cliente.
- [ ] Al confirmar, se crea o actualiza el cliente.

**Reglas**: RN-CLI-01 a RN-CLI-03, RN-PED-10

### US-007 — Ver el historial y reimprimir
**Como** operador
**Quiero** ver los pedidos por día operativo y reimprimir cualquiera
**Para** reponer una comanda perdida

**Criterios de aceptación**:
- [ ] El historial muestra por defecto el día operativo actual, con número, hora, nombre, total y estado.
- [ ] Se puede cambiar de fecha y buscar por número o teléfono.
- [ ] "Reimprimir" usa los datos congelados del pedido.
- [ ] El detalle muestra ítems, cantidades, precios y total.
- [ ] La reimpresión queda registrada.

**Reglas**: RN-COM-06, RN-COM-07, RN-ANU-02

### US-008 — Anular un pedido
**Como** operador
**Quiero** anular un pedido indicando el motivo
**Para** corregir un error sin borrar el registro

**Criterios de aceptación**:
- [ ] Anular pide confirmación y motivo opcional.
- [ ] El pedido queda en estado `anulado` con fecha y hora; no desaparece del historial.
- [ ] Un pedido anulado no se puede reactivar.
- [ ] La reimpresión de un anulado se marca visualmente como "ANULADO".

**Reglas**: RN-ANU-01 a RN-ANU-03

### US-009 — Copia de seguridad
**Como** dueño
**Quiero** que la base se copie sola al cerrar y al abrir el día, más un botón manual
**Para** no perder el historial si la PC se apaga de golpe

**Criterios de aceptación**:
- [ ] Al cerrar la app se intenta una copia y se registra el resultado.
- [ ] En la primera apertura de cada día operativo se hace una copia de apertura, una sola vez por fecha.
- [ ] El botón "Copiar ahora" ejecuta una copia manual.
- [ ] Se conservan 30 días y se eliminan las más viejas solo después de una copia exitosa.
- [ ] Una copia fallida no bloquea el uso de la app ni borra copias previas.
- [ ] La carpeta destino es configurable.

**Reglas**: RN-SEG-01 a RN-SEG-07

## Épica 2 — Beta, segunda iteración

### US-010 — Opciones configurables por producto
**Como** dueño
**Quiero** definir opciones por producto (tipo de pan, tipo de queso, sin ingrediente)
**Para** que el operador las elija con el teclado y salgan impresas

**Criterios de aceptación**:
- [ ] Se definen opciones y valores por producto en el editor de menú.
- [ ] Al agregar un ítem se ofrecen solo las opciones de ese producto.
- [ ] Las opciones requeridas no permiten agregar el ítem sin elegir.
- [ ] Lo elegido se imprime en la comanda y se guarda como snapshot.
- [ ] Sin precio adicional (hasta confirmación del cliente).

**Reglas**: RN-MEN-07

### US-011 — Corregir un pedido con registro
**Como** operador
**Quiero** corregir un pedido y que se reimprima automáticamente
**Para** arreglar un error manteniendo trazabilidad

**Criterios de aceptación**:
- [ ] La corrección guarda snapshot antes/después y motivo.
- [ ] Se reimprime automáticamente tras guardar.
- [ ] El historial indica que el pedido fue corregido.
- [ ] El total se recalcula y se guarda.

**Reglas**: RN-ANU-04, RN-PED-01, RN-PED-03

## Épica 3 — Versión siguiente

### US-012 — Exportar a Excel
Fuera de la beta. Se documentará en su propio change.
