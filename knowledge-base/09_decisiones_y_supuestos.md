# 09 — Decisiones y Supuestos

## Decisiones documentadas

### DD-01 — Stack: C# + .NET 10 LTS + WPF
**Decisión**: WPF con MVVM sobre .NET 10 LTS, SQLite con SQL explícito, xUnit.
**Contexto**: había que elegir stack con dos requisitos que tiran en contra: ≤150 MB de RAM medida como suma privada de todos los procesos, e impresión silenciosa por spooler sin depender de un modelo de impresora.
**Alternativas consideradas**: Tauri (25/55), Electron (36/55), .NET + WPF (47/55), Flutter (29/55), Python + Qt (39/55), con pesos 3/3/2/2/1 sobre RAM, impresión, productividad, costo de desarrollo e instalador.
**Justificación**: WPF es monoproceso (el único con margen real para 150 MB) y `System.Printing` resuelve la impresión silenciosa sin FFI ni plugins. El costo de toolchain (~1 GB, builds de segundos) es el segundo más bajo, crítico en una máquina de 2 núcleos con 0.3 GB de RAM libre.
**Trade-offs aceptados**: XAML tiene curva de aprendizaje; la UI por defecto es anticuada; Windows-only (que es el target); no hay ecosistema web para la UI.

### DD-02 — Impresión: driver de Windows como camino principal, ESC/POS como respaldo, detrás de una interfaz
**Decisión**: `IComandaPrinter` con tres implementaciones: `WindowsDriverComandaPrinter` (default), `EscPosRawComandaPrinter` (respaldo), `FileComandaPrinter` (desarrollo/tests). La elección vive en `impresion.modo`.
**Contexto**: la impresora real no se puede probar antes de la entrega y no se debe programar contra un modelo.
**Alternativas consideradas**: solo driver (riesgo si el driver rasteriza mal); solo ESC/POS (requiere conocer el modelo, viola el brief); `window.print()` (abre diálogo, descartado).
**Justificación**: el driver funciona con la impresora que el cliente ya tiene instalada y respeta el ancho de papel; el respaldo cubre el caso de un driver deficiente sin reescribir el sistema.
**Trade-offs aceptados**: el camino principal no produce bytes comparables, así que se testea el layout, no el flujo del dispositivo; el respaldo agrega trabajo de codificación ESC/POS.

### DD-03 — Día operativo con hora de corte configurable
**Decisión**: la numeración de comandas y el disparador de la copia diaria usan `fecha_operativa = date(fecha_hora_local − hora_corte)`, con corte default `06:00`.
**Contexto**: el local puede atender pasada la medianoche; con día calendario, un pedido a las 01:00 tomaría el número 1 de un día nuevo y rompería la continuidad que se canta en el local.
**Alternativas consideradas**: día calendario (simple, incorrecto para el negocio); turnos fijos (rígido, requiere definir turnos).
**Justificación**: un solo parámetro resuelve el caso sin introducir turnos. Cambiarlo no renumera el pasado.
**Trade-offs aceptados**: hay que documentar que un pedido a las 05:59 pertenece al día anterior; el operador debe entender la lógica.

### DD-04 — Precio congelado por ítem
**Decisión**: `pedido_items` guarda código, nombre y precio al momento de cargar.
**Contexto**: el dueño va a cambiar precios; reimprimir una comanda vieja no puede mostrar otro total.
**Alternativas consideradas**: recalcular con el menú actual (rompe la contabilidad y la confianza en el historial).
**Justificación**: el historial es la fuente de verdad de lo cobrado.
**Trade-offs aceptados**: duplicación de datos y un `JOIN` menos en el historial; se asume como costo correcto.

### DD-05 — Anular no borra
**Decisión**: `pedidos.estado` (`confirmado`/`anulado`) + `anulado_at` + `anulado_motivo`. No existe borrado físico.
**Contexto**: un pedido anulado suele ser un error de carga; borrarlo impide auditar y explicar diferencias de caja.
**Justificación**: trazabilidad con costo mínimo.
**Trade-offs aceptados**: la tabla crece; el historial necesita filtros por estado.

### DD-06 — Producto + presentaciones con código propio
**Decisión**: `productos` 1–N `presentaciones`; cada presentación tiene código de 4 dígitos único y precio propio.
**Contexto**: el brief pide "menú con códigos de 4 dígitos" y "productos con presentaciones", y la carga debe ser de un solo código.
**Alternativas consideradas**: un código por producto + elegir presentación (más lento); presentaciones planas sin producto padre (imposible agrupar y reportar).
**Justificación**: máxima velocidad de carga y modelo limpio para reportes futuros.
**Trade-offs aceptados**: más tablas y un editor de menú con dos niveles.

### DD-07 — Copia de seguridad con tres disparadores
**Decisión**: al cerrar, en la primera apertura de cada día operativo y manual; retención de 30 días.
**Contexto**: "al cerrar" puede no ejecutarse nunca si la PC se apaga de golpe o la app queda abierta todo el día.
**Alternativas consideradas**: solo manual (no sirve); copia cada X minutos (más archivos y complejidad).
**Justificación**: cubre los dos modos de falla reales (cierre abrupto y sesión larga) con un solo disparador extra.
**Trade-offs aceptados**: puede haber dos copias por día; se acepta por seguridad.

### DD-08 — Sin precio adicional en las opciones
**Decisión**: `producto_opciones` y `opcion_valores` no tienen `precio_extra_centavos`.
**Contexto**: el cliente no confirmó que existan opciones con costo.
**Justificación**: no modelar un supuesto que agrega complejidad a totales y snapshots.
**Trade-offs aceptados**: si el cliente confirma extras, se agrega una columna con migración y se extiende el snapshot.

### DD-09 — Sin automatización de UI en la beta
**Decisión**: se testean ViewModels y layout; no se automatiza la UI con UIA.
**Contexto**: 8 pantallas, una sola máquina lenta, y el valor está en el dominio y la impresión.
**Justificación**: mejor retorno por hora invertida; la UI se valida manualmente con checklist.
**Trade-offs aceptados**: regresiones visuales no se detectan solas.

### DD-10 — Orden de construcción: núcleo primero
**Decisión**: (1) menú, carga por código, total, impresión, historial básico con anulación y reimpresión; (2) opciones configurables por producto y correcciones con snapshot.
**Contexto**: la beta necesita ser usable cuanto antes y la impresión es el riesgo #1.
**Justificación**: el flujo núcleo es el que reemplaza la comanda de papel; las opciones y la auditoría fina agregan valor sin bloquear el uso diario.
**Trade-offs aceptados**: durante la primera iteración, "sin ingrediente" se resuelve con la nota libre, no con opciones estructuradas.

### DD-11 — Impresión silenciosa no se valida con Microsoft Print to PDF
**Decisión**: `Microsoft Print to PDF` queda solo para inspección visual. La validación usa layout golden tests, `FileComandaPrinter` y una impresora local con puerto `FILE:`.
**Contexto**: `Microsoft Print to PDF` siempre abre un diálogo para elegir dónde guardar, así que no prueba el requisito de silencio.
**Justificación**: probar el requisito correcto, no un sustituto que lo contradice.
**Trade-offs aceptados**: la validación final del papel real solo puede ocurrir en el local.

## Supuestos inferidos

### SU-01 — Teléfono opcional en mostrador
**Supuesto**: en mostrador no se exige teléfono; en domicilio se recomienda.
**Origen**: el brief lista el teléfono como dato de carga pero no define obligatoriedad.
**Riesgo si es falso**: pedidos de mostrador sin identificación; se resuelve volviéndolo obligatorio con una regla y un campo marcado.
**Cómo validar**: preguntar al cliente si necesita el teléfono siempre.

### SU-02 — Horario estimado precargado en +40 minutos
**Supuesto**: el campo se inicializa con "ahora + 40 min" y es editable.
**Origen**: el brief dice "lo escribe a mano, normalmente 35 a 40 min".
**Riesgo si es falso**: molesta si el operador siempre lo cambia; se ajusta el default o se deja vacío.
**Cómo validar**: observar una jornada de uso.

### SU-03 — La numeración no se reinicia por turno
**Supuesto**: mediodía y noche comparten la secuencia del día operativo.
**Origen**: el brief no menciona turnos.
**Riesgo si es falso**: dos comandas con el mismo número en el mismo día.
**Cómo validar**: preguntar si el local trabaja con turnos y si los números se reinician.

### SU-04 — La comanda no lleva logo ni datos del local
**Supuesto**: la beta imprime solo los datos del pedido.
**Origen**: el brief no lo pide.
**Riesgo si es falso**: el cliente espera ver su marca; se agrega un encabezado configurable.
**Cómo validar**: preguntar antes de la entrega.

### SU-05 — El menú inicial lo carga el dueño
**Supuesto**: la app arranca sin productos y el menú se carga desde el editor.
**Origen**: no existe lista digitalizada de productos, códigos y precios.
**Riesgo si es falso**: la beta no se puede usar el primer día.
**Cómo validar**: pedir la lista en cualquier formato antes de la entrega.

### SU-06 — Una sola persona opera la PC
**Supuesto**: no hace falta distinguir autoría.
**Origen**: el brief dice "una sola PC, sin login".
**Riesgo si es falso**: no se puede saber quién anuló o corrigió.
**Cómo validar**: confirmar con el cliente.
