# Brief de cliente — App de pedidos para Vargas Pizza (v4)

## 1. Contexto
- Cliente: Vargas Pizza (el dueño es el padre del desarrollador)
- Hoy: recibe pedidos por WhatsApp, los anota en una comanda física y
  después los carga a mano en un Excel.
- Problema: doble carga (papel + Excel), cuentas manuales y tiempo.
- Entrega: app de escritorio para Windows, 100% offline, en la PC del local.
- Prioridad del cliente: una beta lo antes posible para empezar a usarla,
  e ir sumando funciones y correcciones después.

## 2. Usuarios
- Quien toma los pedidos en el local. Una sola PC, sin login.

## 3. Funcionalidades (primera beta)
1. **Carga de pedido:** nombre, teléfono, tipo (mostrador/domicilio),
   dirección (si es domicilio), productos por código, horario estimado
   de entrega (lo escribe a mano, normalmente 35 a 40 min), forma de
   pago (efectivo o transferencia).
2. **Menú con códigos de 4 dígitos:** se escribe solo el código.
   Productos con presentaciones: empanadas por unidad, media docena y
   docena (y surtida con precio propio); lomo pizza completo o mitad;
   papas en 2 tamaños. Cada presentación tiene su propio código de
   4 dígitos y cuelga de un producto, así se carga escribiendo un solo
   código y el editor de menú las muestra agrupadas.
3. **Opciones por ítem:** "sin ingrediente" (propuesta: nota libre por
   ítem que sale en la comanda), tipo de pan en lomos, tipo de queso en
   hamburguesas.
4. **Total automático** que se actualiza al agregar productos. Sin costo
   de envío. El precio de cada ítem queda congelado al momento de
   cargarlo: cambiar precios del menú no altera pedidos ya cargados.
5. **Comanda impresa automáticamente** (térmica 80 mm) con: número de
   comanda, hora de entrada, nombre, teléfono, domicilio, código +
   producto, precio de cada ítem y total.
6. **Editor de menú:** crear, editar y quitar productos, precios y códigos.
7. **Clientes guardados:** al escribir el teléfono se carga nombre y
   último domicilio. El domicilio es editable en cada pedido.
8. **Historial de pedidos** guardado, con posibilidad de **corregir o
   anular** un pedido.
9. **Copia de seguridad** local en una carpeta "Copia de seguridad":
   automática al cerrar la app, automática en la primera apertura de
   cada día y manual con un botón.

## 4. Reglas de negocio
- **Numeración de comanda:** correlativa diaria, reinicia a 1 cada día.
- **Anular un pedido no lo borra:** queda en el historial marcado como
  anulado, con fecha y hora.
- **Corregir un pedido lo reimprime automáticamente** y deja registro de
  la corrección.
- **Precio congelado:** cada ítem guarda el precio vigente al momento de
  cargarlo.
- **Total:** suma de los ítems. Sin costo de envío.
- **Copia de seguridad:** automática al cerrar, automática en la primera
  apertura de cada día y manual. Carpeta configurable, retención de
  30 días.
- **Si la impresión falla**, el pedido se guarda igual y la app ofrece
  reimprimirlo.

## 5. Requisitos no funcionales
- **Offline total**, datos en SQLite local.
- **Rendimiento:** Celeron N4020 1.10 GHz, 4 GB de RAM, conviviendo con
  WhatsApp. Metas medibles:
  - Arranque en frío ≤ 3 s (primera ejecución tras reiniciar Windows,
    hasta que la pantalla acepta el primer código).
  - RAM ≤ 150 MB: suma de la memoria privada (Working Set privado) de
    todos los procesos de la app, incluido el webview si el stack usa
    uno, con un pedido cargado y 10 minutos de uso normal. Se mide en
    la PC del local o una equivalente.
  - Alta de un ítem por código < 150 ms.
  - Impresión de una comanda < 2 s.
- **Carga rápida por teclado**, sin depender del mouse (propuesta).
- **Windows:** Windows 10 (confirmado por el cliente).

## 6. Hardware e integraciones
- Impresión en la impresora térmica de 80 mm instalada en Windows,
  elegida desde una pantalla de configuración (modelo de referencia:
  Ocom OCPP-812, USB + LAN, ya instalada y en uso en la PC del local).
  Botón de impresión de prueba y de reimpresión por pedido. La app no se
  programa contra el modelo: imprime a través del sistema operativo, en
  la impresora que se elija.
- Ancho de papel y tamaño de letra de la comanda configurables.
- WhatsApp: sin integración, el pedido se carga a mano.

## 7. Alcance
- **Beta:** puntos 1 a 9.
- **Versión siguiente:** exportación a Excel.
- **Fuera de alcance por ahora:** integración con WhatsApp, cierre de
  caja, stock, reportes, vuelto.

## 8. Entrega
- Instalador `.exe` generado con GitHub Actions. Actualizaciones manuales.
- Acuerdo comercial: [PENDIENTE]

## 9. Preguntas abiertas
- ¿Las pizzas tienen distintos tamaños? Si sí, ¿cada tamaño lleva su
  propio código y precio, o hay una regla de precio por tamaño?
- ¿Se piden pizzas mitad y mitad de dos variedades? Si sí, ¿cómo se
  codifica, cómo se calcula el precio y cómo se imprime en la comanda?
- Empanada surtida: ¿qué variedades lleva y se detalla en la comanda, o
  se imprime solo "surtida"?
