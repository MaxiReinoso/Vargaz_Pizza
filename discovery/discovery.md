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
8. **Respaldar**: como dueño, quiero que la base se copie sola al cerrar la app y en la primera apertura de cada día, más un botón manual, para no perder el historial si la PC falla o se apaga de golpe.

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
9. **Copia de seguridad local** automática al cerrar, automática en la primera apertura de cada día y con botón manual.

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
- **Copia de seguridad**: automática al cerrar la app, automática en la primera apertura de cada día y manual desde configuración. Retención de 30 días. La copia de apertura cubre el caso de una PC que se apaga de golpe o una app que queda abierta todo el día, donde el cierre nunca se ejecuta.
- **Sin login**: un solo usuario implícito.

## 8. Integraciones

- **Spooler de impresión de Windows**: la única integración. La app imprime en la impresora instalada y elegida en la pantalla de configuración, de forma silenciosa, sin diálogo por pedido.
- **Sin nube, sin WhatsApp, sin APIs de terceros, sin telemetría.**
- **Impresora real confirmada**: el cliente confirma que la térmica Ocom OCPP-812 ya está instalada y configurada en la PC del local. **No se programa contra ese modelo ni contra ningún otro**: se imprime a través del spooler de Windows, contra la impresora que el usuario elija. La validación final es el botón *Imprimir prueba* el día de la instalación.
- **Microsoft Print to PDF no sirve para validar la impresión silenciosa**: siempre abre un diálogo para elegir dónde guardar el PDF. Sirve para inspeccionar visualmente el diseño de la comanda, no para probar que no hay diálogo.
- **Estrategia de prueba de impresión** (se detalla en la estrategia de pruebas de la KB):
  1. **Tests del render**: la comanda se genera a bytes (texto/ESC-POS o XPS) y se testea el contenido: ancho de 80 mm, tamaño de letra, wrapping de nombres largos, precio por ítem y total. No requiere impresora.
  2. **Impresión silenciosa real contra una impresora local de prueba** con puerto `FILE:` o driver *Generic / Text Only* apuntando a un archivo. Windows imprime sin diálogo y deja un artefacto inspeccionable. Ejercita el mismo camino Win32 que la impresora real.
  3. **Modo desarrollo explícito** que escribe en un archivo el flujo exacto de bytes que se envió a la impresora, para tests automatizados y para comparar antes/después de un cambio.
  4. **Validación en el local**: *Imprimir prueba* sobre la Ocom OCPP-812 con ancho y tamaño de letra configurados, reimpresión de un pedido, y prueba de error (apagar la impresora y verificar que el pedido se guarda igual y aparece la alerta con Reimprimir).

## 9. Restricciones

- **100% offline**, datos locales en SQLite. Sin servidores ni servicios en la nube.
- **Hardware del local**: Intel Celeron N4020 1.10 GHz, 4 GB de RAM, **Windows 10 confirmado por el cliente**. La PC corre WhatsApp al mismo tiempo.
- **Rendimiento como requisito de diseño**, no como optimización posterior. Metas medibles y cómo se miden:
  - **Arranque en frío ≤ 3 s**: desde el doble clic hasta que la pantalla principal acepta el primer código. "Frío" = primera ejecución después de reiniciar Windows. Tres mediciones, se toma la mediana.
  - **RAM en uso normal ≤ 150 MB**: **suma del Working Set PRIVADO (memoria privada) de TODOS los procesos de la app**, incluidos los subprocesos del webview y cualquier proceso auxiliar que el stack levante. Se usa el Working Set privado y no el total porque el total cuenta dos veces las páginas compartidas entre procesos (DLLs, runtime) y hace que la suma no represente memoria realmente consumida. Se mide con la app abierta, un pedido cargado y 10 minutos de uso normal: no en reposo sobre una pantalla vacía. Se mide en la PC del local o en una equivalente.
  - **Alta de un ítem por código < 150 ms**: desde Enter sobre el código hasta que el ítem aparece en la lista y el total se actualiza. Mediana de 20 altas.
  - **Impresión de una comanda < 2 s**: desde confirmar el pedido hasta que Windows reporta el trabajo entregado al spooler. No incluye el tiempo físico de impresión del papel.
- **Costo de desarrollo en la máquina del desarrollador** (medido, no estimado): AMD 3020e, 2 núcleos a 1.2 GHz, 5.9 GB de RAM usables con **0.3 GB libres** en uso normal, HDD de 2 TB con ~1032 GB libres, Windows 10 Pro for Workstations x64. Instalado: Node 24.20.0, npm 12.0.2, git 2.55, Python 3.13.7. **No instalado**: Rust/cargo, .NET SDK, Flutter, Visual Studio Build Tools, cmake, gcc. Consecuencias a evaluar al elegir stack: tamaño de las herramientas a instalar, tiempo y consumo de RAM de cada compilación, y velocidad del ciclo de prueba (editar → ver cambio). Un stack que exija compilaciones largas o más de 2-3 GB de toolchain tiene un costo real en esta máquina.
- **Impresión**: térmica de 80 mm, silenciosa, sin diálogo por pedido, con "Imprimir prueba" en configuración y "Reimprimir" por pedido. Ancho de papel y tamaño de letra configurables. En desarrollo se valida con tests del render + una impresora local de prueba con puerto `FILE:` (silenciosa); *Microsoft Print to PDF* solo sirve para inspección visual porque abre diálogo. La validación final es en el local, con *Imprimir prueba* sobre la impresora ya instalada.
- **Entrega**: instalador `.exe` para Windows generado con GitHub Actions al crear un tag, publicado en GitHub Releases. Sin firma de código.
- **Desarrollador principiante** que trabaja con agentes de IA: el stack debe ser productivo, con buena documentación y errores comprensibles.
- **Fecha de entrega**: no definida formalmente; la prioridad es una beta usable lo antes posible.

## 10. Riesgos

| Riesgo | Tipo | Mitigación propuesta |
|---|---|---|
| Impresión silenciosa desde una app de escritorio en Windows (drivers, spooler, PDF virtual) | Supuesto sin probar — **riesgo #1** | Primer change del roadmap: flujo de impresión de punta a punta. Se valida con tests del render + impresora local de prueba con puerto `FILE:`, que imprime sin diálogo. *Microsoft Print to PDF* no sirve para esto: siempre abre diálogo para elegir el archivo. |
| Rendimiento real en el Celeron N4020 conviviendo con WhatsApp | Supuesto sin probar | Elegir stack liviano desde el diseño y medir con la app real en la PC del local (o la del desarrollador, que es equivalente) con la definición de medición acordada. |
| El instalador sin firma dispara SmartScreen ("Windows protegió tu PC") | Hecho conocido | Documentar el paso a paso para el cliente; evaluar firma en el futuro. |
| Curva de aprendizaje del stack si incluye Rust | Supuesto | Evaluar explícitamente el costo de aprendizaje al elegir el stack, no solo el consumo de RAM. Rust no está instalado en la máquina del desarrollador. |
| El menú real (productos, códigos, precios) no está digitalizado | Dato faltante | Cargarlo con el editor de menú antes de la beta; pedir la lista en cualquier formato al cliente. |
| La impresora real está en el local, no en el equipo de desarrollo | Restricción | La app se diseña contra el spooler de Windows, no contra el modelo. La validación final es *Imprimir prueba* el día de la instalación, con la impresora que ya está en uso. |
| Máquina de desarrollo con 2 núcleos, 6 GB y ~0.3 GB de RAM libre | Restricción operativa | Elegir un stack con toolchain chico, compilación incremental rápida y pocos procesos en segundo plano; medir el ciclo de prueba antes de comprometerse. |
| Corte de luz en medio de un pedido | Supuesto sin probar | Guardado inmediato en SQLite y recuperación del borrador al abrir. |
| La PC del local es también la PC de WhatsApp: alguien puede tocar la base o borrar archivos | Riesgo operativo | Copia de seguridad automática al cerrar y al abrir el día + carpeta de backup separada. |

## 11. Preguntas abiertas

**Resuelto**: la impresora térmica (Ocom OCPP-812) ya está instalada y configurada en la PC del local, según el cliente. Sigue vigente que **no se programa contra el modelo**: se imprime vía spooler de Windows, contra la impresora elegida en configuración, y la validación final es *Imprimir prueba* el día de la instalación.

- **¿El cliente tiene la lista de productos, códigos y precios** en algún formato (papel, Excel, foto) para cargar el menú inicial?
- **¿Las pizzas tienen distintos tamaños?** Si sí, cada tamaño necesita su propio código y precio, o hay una regla de precio por tamaño que haya que modelar.
- **¿Se piden pizzas mitad y mitad de dos variedades?** Si sí: cómo se codifica, cómo se calcula el precio (¿promedio, la más cara, precio propio?) y cómo se imprime en la comanda.
- **Empanada surtida**: ¿qué variedades lleva y se detalla en la comanda, o se imprime solo "surtida"?
- **¿Qué pasa si se corta la luz en medio de un pedido?** Supuesto: guardado inmediato + recuperación del borrador. Confirmar con el cliente si le sirve recuperar el pedido o prefiere empezar de cero.
- **Acuerdo comercial** `[PENDIENTE]`: no bloquea el desarrollo, pero conviene cerrarlo antes de la entrega.
- **¿Hay más de una persona que carga pedidos** en la misma PC? Confirmado que no hay login; solo importa si en el futuro hay que distinguir autoría.
- **¿El cliente quiere el nombre/logo del local en la comanda?** En la beta se imprime solo el pedido.

---

## Datos confirmados por el cliente

- **Sistema operativo**: Windows 10 en la PC del local.
- **Impresora**: la térmica Ocom OCPP-812 ya está instalada y en uso en la PC del local. No se programa contra el modelo: se imprime vía spooler de Windows.

## Supuestos tomados para poder avanzar

Estos puntos no estaban decididos en el brief y se resolvieron con un supuesto razonable. Están marcados para que el cliente los confirme:

1. **Teléfono opcional en mostrador**, recomendado en domicilio.
2. **Horario estimado precargado** en "ahora + 40 min", siempre editable.
3. **La numeración diaria no se reinicia por turno.**
4. **Forma de pago se imprime** en la comanda; no hay cálculo de vuelto.
5. **La comanda no lleva logo ni datos del local** en la beta.
6. **Metas de rendimiento**: ≤ 3 s de arranque, ≤ 150 MB de RAM, < 150 ms por ítem, < 2 s por impresión, con las definiciones de medición de la sección 9.

## Decisiones tomadas en Discovery

Estas sí quedaron decididas por el desarrollador en esta fase:

1. **Códigos por presentación agrupados bajo un producto** (producto + variantes con precio y código propios).
2. **Comanda correlativa diaria**; anular no borra, marca; corregir reimprime y deja registro.
3. **Si la impresión falla, el pedido se guarda igual** y se ofrece Reimprimir.
4. **Opciones por ítem**: nota libre + opciones configurables por producto desde el editor de menú.
5. **Carga teclado primero** como requisito, no como nice-to-have.
6. **El pedido congela el precio** del ítem al momento de cargarlo.
7. **Copia de seguridad**: automática al cerrar la app, automática en la primera apertura de cada día y manual con botón. Carpeta configurable (default `Documentos\Vargas Pizza\Copia de seguridad`), retención de 30 días.
8. **Sin análisis de competidores**.
