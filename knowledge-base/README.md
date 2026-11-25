# Vargas Pizza — Base de Conocimiento

Base de conocimiento del proyecto, generada a partir de `docs/discovery/brief-clientes.md` (v4), `discovery/discovery.md` y las decisiones de fundación aprobadas.

## Índice de Archivos

| Archivo | Contenido |
|---|---|
| [01_vision_y_objetivos.md](01_vision_y_objetivos.md) | Propósito, objetivos, alcance de la beta, métricas medibles |
| [02_descripcion_general.md](02_descripcion_general.md) | Stack, arquitectura general, única integración, impresión |
| [03_actores_y_roles.md](03_actores_y_roles.md) | Un solo actor, sin login, matriz de permisos |
| [04_modelo_de_datos.md](04_modelo_de_datos.md) | Entidades, ERD, constraints, seed, día operativo |
| [05_reglas_de_negocio.md](05_reglas_de_negocio.md) | Reglas codificadas RN-XX-NN por dominio |
| [06_funcionalidades.md](06_funcionalidades.md) | 12 historias de usuario por épica y criterios de aceptación |
| [07_flujos_principales.md](07_flujos_principales.md) | 6 flujos extremo a extremo con casos de error |
| [08_arquitectura_propuesta.md](08_arquitectura_propuesta.md) | Patrones, carpetas, impresión, pruebas, CI/CD |
| [09_decisiones_y_supuestos.md](09_decisiones_y_supuestos.md) | DD-01 a DD-11 y SU-01 a SU-06 |
| [10_preguntas_abiertas.md](10_preguntas_abiertas.md) | Inconsistencias y preguntas priorizadas |

## Quick Start para Desarrolladores

1. Entender el dominio → [01](01_vision_y_objetivos.md), [03](03_actores_y_roles.md)
2. Entender los datos → [04](04_modelo_de_datos.md)
3. Entender las reglas → [05](05_reglas_de_negocio.md)
4. Entender la arquitectura y las pruebas → [02](02_descripcion_general.md), [08](08_arquitectura_propuesta.md)
5. Implementar → [07](07_flujos_principales.md), [06](06_funcionalidades.md)
6. Antes de codificar → [10](10_preguntas_abiertas.md)

## Resumen Ejecutivo

App de escritorio Windows 10, 100 % offline, para tomar pedidos de una pizzería: carga por código de 4 dígitos, total automático e impresión silenciosa de la comanda en una térmica de 80 mm.

Stack: C# + .NET 10 LTS + WPF (MVVM) + SQLite + xUnit. La impresión usa el driver de Windows como camino principal y ESC/POS crudo como respaldo, siempre detrás de `IComandaPrinter`.

El orden de construcción pone primero el flujo núcleo (menú, carga, total, impresión, historial básico) porque la impresión silenciosa es el mayor riesgo técnico del proyecto.
