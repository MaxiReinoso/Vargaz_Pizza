# 01 — Visión y Objetivos

## Propósito del sistema

Vargas Pizza es una app de escritorio para Windows que reemplaza la comanda de papel: quien atiende carga el pedido una sola vez, el total se calcula solo y la comanda se imprime automáticamente en la impresora térmica de 80 mm que ya está instalada en el local.

Hoy el pedido llega por WhatsApp, se anota a mano y después se vuelve a cargar en un Excel. Eso produce doble carga, cuentas manuales y tiempo perdido en el momento de mayor presión. La app elimina la segunda carga y el cálculo manual.

## Objetivos por actor

| Actor | Objetivo principal | Objetivos secundarios |
|---|---|---|
| Operador (quien atiende) | Cargar un pedido completo en menos de un minuto, sin mouse | Ver el total actualizarse solo · imprimir la comanda sin tocar nada · reimprimir · buscar clientes por teléfono · corregir y anular |
| Dueño del local (mismo actor, sin rol separado) | Mantener el menú y los precios al día sin depender del desarrollador | Tener historial confiable · contar con copia de seguridad automática |

No hay más actores. No hay login, ni permisos, ni clientes finales como usuarios del sistema: el cliente sigue pidiendo por WhatsApp.

## Alcance v0.1 (beta)

Las funciones 1 a 9 del brief:

1. Carga de pedido: nombre, teléfono, tipo (mostrador/domicilio), dirección, productos por código, horario estimado, forma de pago.
2. Menú con códigos de 4 dígitos por presentación, agrupados bajo un producto.
3. Opciones por ítem: nota libre impresa + opciones configurables por producto desde el editor de menú.
4. Total automático, sin costo de envío.
5. Comanda impresa automáticamente en térmica de 80 mm.
6. Editor de menú: productos, presentaciones, precios y códigos.
7. Clientes guardados por teléfono con último domicilio editable.
8. Historial de pedidos con anulación y reimpresión.
9. Copia de seguridad local automática y manual.

**Orden de construcción acordado**: primero el flujo núcleo (menú, carga por código, total, impresión, historial básico con anulación y reimpresión). Las opciones configurables por producto y las correcciones con snapshot se construyen después, dentro de la misma beta.

## Fuera de alcance

- Exportación a Excel (primera función de la versión siguiente).
- Integración con WhatsApp: el pedido se sigue cargando a mano.
- Cierre de caja, control de stock, reportes y cálculo de vuelto.
- Multiusuario, login, permisos, sincronización o nube.
- Actualizaciones automáticas: la entrega es un instalador manual.
- Logo o datos del local en la comanda (pendiente de confirmación del cliente).

## Métricas de éxito

| Métrica | Objetivo | Cómo se mide |
|---|---|---|
| Arranque en frío | ≤ 3 s | Primera ejecución tras reiniciar Windows, hasta que la pantalla acepta el primer código. Mediana de 3. |
| Memoria | ≤ 150 MB | Suma del Working Set privado de todos los procesos de la app, con un pedido cargado y 10 minutos de uso. |
| Alta de ítem por código | < 150 ms | Desde Enter hasta ítem visible y total actualizado. Mediana de 20. |
| Impresión | < 2 s | Desde confirmar hasta que Windows reporta el trabajo entregado al spooler. |
| Carga de un pedido típico | < 60 s | Cronómetro en el local con un pedido de 4 ítems. |
| Pérdida de datos | 0 pedidos | Historial íntegro después de un corte de luz y de una restauración de backup. |
