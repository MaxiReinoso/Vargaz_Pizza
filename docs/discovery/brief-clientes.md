# Brief de cliente — App de pedidos para Vargas Pizza (v3, borrador)

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
   papas en 2 tamaños.
3. **Opciones por ítem:** "sin ingrediente" (propuesta: nota libre por
   ítem que sale en la comanda), tipo de pan en lomos, tipo de queso en
   hamburguesas.
4. **Total automático** que se actualiza al agregar productos. Sin costo
   de envío.
5. **Comanda impresa automáticamente** (térmica 80 mm) con: número de
   comanda, hora de entrada, nombre, teléfono, domicilio, código +
   producto, precio de cada ítem y total.
6. **Editor de menú:** crear, editar y quitar productos, precios y códigos.
7. **Clientes guardados:** al escribir el teléfono se carga nombre y
   último domicilio. El domicilio es editable en cada pedido.
8. **Historial de pedidos** guardado, con posibilidad de **corregir o
   anular** un pedido.
9. **Copia de seguridad** local en una carpeta "Copia de seguridad".

## 4. Requisitos no funcionales
- **Offline total**, datos en SQLite local.
- **Rendimiento:** Celeron N4020 1.10 GHz, 4 GB de RAM, conviviendo con
  WhatsApp. Se define desde el stack. Meta medible: [COMPLETAR, ej. abrir
  en menos de X seg y consumir menos de X MB].
- **Carga rápida por teclado**, sin depender del mouse (propuesta).
- **Windows:** [PREGUNTAR versión]

## 5. Hardware e integraciones
- Impresión en la impresora térmica de 80 mm instalada en Windows, elegida
  desde una pantalla de configuración (modelo de referencia: Ocom OCPP-812,
  USB + LAN). Botón de impresión de prueba y de reimpresión por pedido.
- Ancho de papel y tamaño de letra de la comanda configurables.
- WhatsApp: sin integración, el pedido se carga a mano.

## 6. Alcance
- **Beta:** puntos 1 a 9.
- **Versión siguiente:** exportación a Excel.
- **Fuera de alcance por ahora:** integración con WhatsApp, cierre de
  caja, stock, reportes, vuelto.

## 7. Entrega
- Instalador `.exe` generado con GitHub Actions. Actualizaciones manuales.
- Acuerdo comercial: [PENDIENTE]