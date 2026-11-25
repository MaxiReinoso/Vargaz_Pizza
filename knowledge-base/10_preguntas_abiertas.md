# 10 — Preguntas Abiertas

## Inconsistencias detectadas

### IN-01 — Las opciones por producto figuran como "(propuesta)" en el brief
**Documento A dice** (`docs/discovery/brief-clientes.md`, función 3): "sin ingrediente (**propuesta**: nota libre por ítem…), tipo de pan en lomos, tipo de queso en hamburguesas".
**Documento B dice** (`discovery/discovery.md`, decisión 4): nota libre + opciones configurables por producto, tomado como decisión del desarrollador.
**Impacto**: la épica 2 (US-010) no debería construirse hasta que el cliente confirme que quiere opciones estructuradas y no solo nota libre.
**Resolución propuesta**: validar con el cliente en la primera visita. Mientras tanto, la beta núcleo usa nota libre y no bloquea nada.

### IN-02 — La numeración cambia de "día calendario" a "día operativo"
**Documento A dice** (brief v4): "correlativa diaria, reinicia a 1 cada día".
**Documento B dice** (DD-03): correlativa por día operativo con hora de corte configurable (default 06:00).
**Impacto**: ninguno funcional; es una precisión. Un pedido a las 01:00 pertenece al día anterior.
**Resolución propuesta**: el brief y el discovery se actualizan para decir "día operativo (corte 06:00)". Queda pendiente que el cliente entienda la lógica en la entrega.

### IN-03 — Ancho de papel: "80 mm" fijo vs configurable
**Documento A dice**: térmica de 80 mm.
**Documento B dice**: ancho de papel configurable.
**Impacto**: ninguno; 80 mm es el default y lo configurable es la excepción.
**Resolución propuesta**: `impresion.ancho_papel_mm` default `80`.

## Preguntas abiertas (priorizadas)

| Prioridad | Pregunta | Bloquea | Decisor |
|---|---|---|---|
| **Alta** | ¿Las pizzas tienen distintos tamaños? Si sí, ¿cada tamaño lleva código y precio propios o hay una regla de precio por tamaño? | Modelo de menú y carga inicial | Cliente |
| **Alta** | ¿Se piden pizzas mitad y mitad de dos variedades? ¿Cómo se cobra (promedio, la más cara, precio propio)? | Modelo de ítem (componentes por ítem) | Cliente |
| **Alta** | Empanada surtida: ¿qué variedades lleva y se detalla en la comanda o se imprime solo "surtida"? | Carga inicial del menú y layout de comanda | Cliente |
| **Alta** | ¿Existe una lista de productos, códigos y precios en algún formato (papel, Excel, foto)? | Uso real de la beta el primer día | Cliente |
| **Media** | ¿Se confirman las opciones configurables por producto (tipo de pan, tipo de queso, sin ingrediente)? | Épica 2 / US-010 | Cliente |
| **Media** | ¿Hay opciones que cambian el precio (ej. agregado de queso)? | Migración de `opcion_valores` | Cliente |
| **Media** | Si se corta la luz en medio de un pedido, ¿conviene recuperar el borrador o empezar de cero? | Guardado del borrador | Cliente |
| **Media** | ¿La Ocom OCPP-812 acepta ESC/POS crudo? | Solo si hay que activar el camino de respaldo | Equipo técnico (se valida en el local) |
| **Baja** | ¿El cliente quiere el nombre o logo del local en la comanda? | Encabezado configurable | Cliente |
| **Baja** | ¿Trabaja con turnos que reinician la numeración? | RN-COM-01 / SU-03 | Cliente |
| **Baja** | ¿Más de una persona carga pedidos en la misma PC? | Autoría futura | Cliente |
| **Baja** | ¿Se cierra el acuerdo comercial antes de la entrega? | Nada técnico; sí la relación | Cliente / desarrollador |

## Bloqueos por fase

- **No bloquea el change de impresión**: todas las preguntas de menú. La comanda de prueba usa datos de ejemplo.
- **Bloquea el change de modelo de datos / menú**: tamaños de pizza, mitad y mitad y variedades de la surtida.
- **Bloquea el uso real de la beta**: la lista de productos, códigos y precios.
