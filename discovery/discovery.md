# Discovery — Vargas Pizza

**Fecha**: 2026-11-25
**Fuentes investigadas**: ninguna URL externa. Fuente principal: `docs/discovery/brief-clientes.md` (v3) + ronda de Discovery con el desarrollador.
**Decisión de fuentes**: no se corre análisis de competidores. Es una herramienta a medida para un solo local; el "competidor" real es el proceso actual (papel + Excel + WhatsApp).

---

## 1. Problema que resuelve

Vargas Pizza recibe pedidos por WhatsApp, los anota en una comanda de papel y después los vuelve a cargar a mano en un Excel. Eso genera doble carga, cuentas hechas a mano y tiempo perdido, justo en el momento de mayor presión: cuando el cliente está esperando.

La app reemplaza la comanda de papel: se carga el pedido una sola vez, el total se calcula solo y la comanda se imprime automáticamente en la térmica de 80 mm.

## 2. Usuarios / roles

- **Quien atiende el mostrador** (único rol): carga pedidos, imprime y reimprime comandas, corrige o anula, edita el menú y hace la copia de seguridad.

Una sola PC, sin login, sin multiusuario, sin permisos diferenciados. No hay roles administrativos ni clientes finales como usuarios del sistema: el cliente sigue pidiendo por WhatsApp.

## 3. Casos de uso

1. **Tomar un pedido**: como quien atiende, quiero cargar nombre, teléfono, tipo (mostrador/domicilio), dirección, productos por código, horario estimado y forma de pago, para tener el pedido listo sin escribir nada a mano dos veces.
2. **Cobrar con total automático**: como quien atiende, quiero que el total se actualice al agregar cada producto, para no hacer cuentas mentales ni en papel.
3. **Imprimir la comanda**: como quien atiende, quiero que al confirmar el pedido se imprima sola en la térmica, para que la cocina la tenga sin que yo toque nada.
4. **Reimprimir**: como quien atiende, quiero reimprimir cualquier pedido del historial, para reponer una comanda perdida o corregir un error.
5. **Buscar cliente por teléfono**: como quien atiende, quiero que al tipear el teléfono se carguen nombre y último domicilio, para no preguntar los datos de nuevo.
6. **Editar el menú**: como dueño, quiero crear, editar y quitar productos, presentaciones, precios y códigos, para mantener la carta al día sin depender del desarrollador.
7. **Corregir o anular**: como quien atiende, quiero corregir o anular un pedido del historial, para arreglar un error sin romper la trazabilidad.
8. **Respaldar**: como dueño, quiero una copia de seguridad local automática, para no perder el historial si la PC falla.

## 4. Competidores / soluciones existentes

No se relevaron competidores comerciales. El análisis se consideró de bajo valor porque:

- El producto es de uso interno de un solo local, no se comercializa.
- El dueño ya tiene un proceso definido; la app lo digitaliza, no compite por un mercado.
- La restricción de 100% offline y una PC de 4 GB descarta de entrada los POS en la nube.

**Referencia conceptual**: los sistemas de punto de venta gastronómico (Fudo, Maxirest, Tango Restô) confirman el patrón de "código de producto → ítem → comanda impresa", pero ninguno se toma como fuente de requisitos. El brief del cliente manda.

## 5. Funcionalidades necesarias (beta = puntos 1 a 9 del brief)

1. **Carga de pedido**: nombre, teléfono, tipo (mostrador/domicilio), dirección si es domicilio, productos por código, horario estimado de entrega (editable, precargado en +40 min) y forma de pago (efectivo/transferencia).
2. **Menú con códigos de 4 dígitos**, con presentaciones agrupadas bajo un producto: empanadas (unidad, media docena, docena, surtida con precio propio), lomo pizza (completo/mitad), papas (2 tamaños).
3. **Opciones por ítem**: nota libre que sale impresa + opciones configurables por producto desde el editor de menú (tipo de pan en lomos, tipo de queso en hamburguesas, "sin ingrediente" en pizzas).
4. **Total automático** al agregar o quitar ítems. Sin costo de envío.
5. **Comanda impresa automáticamente** en térmica de 80 mm: número de comanda, hora de entrada, nombre, teléfono, domicilio, código + producto, precio por ítem y total.
6. **Editor de menú**: alta, edición y baja de productos, presentaciones, precios y códigos.
7. **Clientes guardados**: al escribir el teléfono se autocompletan nombre y último domicilio; el domicilio es editable en cada pedido.
8. **Historial de pedidos** con corrección y anulación.
9. **Copia de seguridad local** automática al cerrar la app + botón manual.

## 6. Funcionalidades opcionales (post-beta)

- Exportación a Excel (primera función de la versión siguiente, ya decidida).
- Integración con WhatsApp (fuera de alcance).
- Cierre de caja, control de stock, reportes, cálculo de vuelto (fuera de alcance).
- Actualizaciones automáticas (la entrega es manual).
- Logo y datos del local en la comanda.

## 7. Reglas de negocio

- **Código por presentación**: cada presentación tiene su propio código de 4 dígitos, pero en el modelo de datos cuelga de un producto padre. Se escribe un código y el ítem entra directo.
- **Precio por presentación**: la surtida tiene precio propio, distinto del de la docena común.
- **Total**: suma de ítems. No hay costo de envío, descuentos ni recargos en la beta.
- **Precio congelado**: el pedido guarda el precio vigente al momento de cargar el ítem. Cambiar precios en el menú no altera pedidos existentes.
- **Numeración de comanda**: correlativa diaria, reinicia a 1 cada día. No se reinicia por turno.
- **Anulación**: el pedido nunca se borra. Queda marcado como anulado con fecha y hora.
- **Corrección**: corregir un pedido lo reimprime automáticamente y deja registro de la corrección.
- **Opciones por producto**: se definen en el editor de menú, no en código.
- **Carga teclado primero**: foco automático en el código, Enter agrega, Tab navega, atajos para confirmar/imprimir.
- **Sin login**: un solo usuario implícito.

## 8. Integraciones

- **Spooler de impresión de Windows**: la única integración. La app imprime en la impresora instalada y elegida en la pantalla de configuración, de forma silenciosa, sin diálogo por pedido.
- **Sin nube, sin WhatsApp, sin APIs de terceros, sin telemetría.**
- Durante el desarrollo se prueba con la impresora virtual *Microsoft Print to PDF*. El modelo de referencia del cliente es Ocom OCPP-812, pero **no se programa contra ese modelo**: no se habla con el hardware, se habla con Windows.

## 9. Restricciones

- **100% offline**, datos locales en SQLite. Sin servidores ni servicios en la nube.
- **Hardware**: Intel Celeron N4020 1.10 GHz, 4 GB de RAM, Windows 10. La PC corre WhatsApp al mismo tiempo.
- **Rendimiento como requisito de diseño**, no como optimización posterior. Meta medible:
  - Arranque en frío ≤ 3 s.
  - RAM en uso normal ≤ 150 MB.
  - Alta de un ítem por código < 150 ms.
  - Impresión de una comanda < 2 s.
- **Impresión**: térmica de 80 mm, silenciosa, sin diálogo por pedido, con "Imprimir prueba" en configuración y "Reimprimir" por pedido. Ancho de papel y tamaño de letra configurables (no se puede probar con la impresora real antes de la entrega).
- **Entrega**: instalador `.exe` para Windows generado con GitHub Actions al crear un tag, publicado en GitHub Releases. Sin firma de código.
- **Desarrollador principiante** que trabaja con agentes de IA: el stack debe ser productivo, con buena documentación y errores comprensibles.
- **Fecha de entrega**: no definida formalmente; la prioridad es una beta usable lo antes posible.

## 10. Riesgos

| Riesgo | Tipo | Mitigación propuesta |
|---|---|---|
| Impresión silenciosa desde una app de escritorio en Windows (drivers, spooler, PDF virtual) | Supuesto sin probar — **riesgo #1** | Primer change del roadmap: flujo de impresión de punta a punta con Microsoft Print to PDF. Si no funciona, se descubre antes de construir el resto. |
| Rendimiento real en el Celeron N4020 conviviendo con WhatsApp | Supuesto sin probar | Elegir stack liviano desde el diseño + medir con la app real en la PC del local antes de la entrega. |
| El instalador sin firma dispara SmartScreen ("Windows protegió tu PC") | Hecho conocido | Documentar el paso a paso para el cliente; evaluar firma en el futuro. |
| Curva de aprendizaje del stack si incluye Rust | Supuesto | Evaluar explícitamente el costo de aprendizaje al elegir el stack, no solo el consumo de RAM. |
| El menú real (productos, códigos, precios) no está digitalizado | Dato faltante | Cargarlo con el editor de menú antes de la beta; pedir la lista en cualquier formato al cliente. |
| La impresora real no está disponible durante el desarrollo | Restricción | Diseñar contra el spooler de Windows, no contra el modelo; validar el día de la entrega con "Imprimir prueba". |
| Corte de luz en medio de un pedido | Supuesto sin probar | Guardado inmediato en SQLite y recuperación del borrador al abrir. |
| La PC del local es también la PC de WhatsApp: alguien puede tocar la base o borrar archivos | Riesgo operativo | Copia de seguridad automática al cerrar + carpeta de backup separada. |

## 11. Preguntas abiertas

- **¿Ya está comprada e instalada la impresora térmica?** La validación final depende de probar con ella. Hasta entonces, *Microsoft Print to PDF*.
- **¿El cliente tiene la lista de productos, códigos y precios** en algún formato (papel, Excel, foto) para cargar el menú inicial?
- **¿Qué pasa si se corta la luz en medio de un pedido?** Supuesto: guardado inmediato + recuperación del borrador. Confirmar con el cliente si le sirve recuperar el pedido o prefiere empezar de cero.
- **Acuerdo comercial** `[PENDIENTE]`: no bloquea el desarrollo, pero conviene cerrarlo antes de la entrega.
- **¿Hay más de una persona que carga pedidos** en la misma PC? Confirmado que no hay login; solo importa si en el futuro hay que distinguir autoría.
- **¿El cliente quiere el nombre/logo del local en la comanda?** En la beta se imprime solo el pedido.

---

## Supuestos tomados para poder avanzar

Estos puntos no estaban decididos en el brief y se resolvieron con un supuesto razonable. Están marcados para que el cliente los confirme:

1. **Windows 10** en la PC del local (el brief decía `[PREGUNTAR]`).
2. **Teléfono opcional en mostrador**, recomendado en domicilio.
3. **Horario estimado precargado** en "ahora + 40 min", siempre editable.
4. **La numeración diaria no se reinicia por turno.**
5. **Forma de pago se imprime** en la comanda; no hay cálculo de vuelto.
6. **La comanda no lleva logo ni datos del local** en la beta.
7. **Metas de rendimiento**: ≤ 3 s de arranque, ≤ 150 MB de RAM, < 150 ms por ítem, < 2 s por impresión.

## Decisiones tomadas en Discovery

Estas sí quedaron decididas por el desarrollador en esta fase:

1. **Códigos por presentación agrupados bajo un producto** (producto + variantes con precio y código propios).
2. **Comanda correlativa diaria**; anular no borra, marca; corregir reimprime y deja registro.
3. **Si la impresión falla, el pedido se guarda igual** y se ofrece Reimprimir.
4. **Opciones por ítem**: nota libre + opciones configurables por producto desde el editor de menú.
5. **Carga teclado primero** como requisito, no como nice-to-have.
6. **El pedido congela el precio** del ítem al momento de cargarlo.
7. **Copia de seguridad automática al cerrar** + botón manual, carpeta configurable (default `Documentos\Vargas Pizza\Copia de seguridad`), retención de 30 días.
8. **Sin análisis de competidores**.
