# CHANGES — Secuencia de Implementación

> Índice canónico de todos los changes del proyecto **Vargas Pizza**.
> Cada change es atómico: un agente puede implementarlo en una sesión (~4-6 horas).
> **Leer este archivo antes de ejecutar cualquier `/opsx:propose`.**

---

## Cómo usar este documento

1. Identificar el change a implementar y verificar que sus dependencias estén archivadas en `openspec/changes/archive/`.
2. Leer los archivos de la knowledge-base indicados en **Leer antes** (con la sección indicada, no el archivo completo a ciegas).
3. Ejecutar `/opsx:propose <nombre-del-change>` usando el nombre kebab-case del change (el ID `C-NN` es solo para este índice).
4. Al terminar, archivarlo con `/opsx:archive <nombre-del-change>`.
5. Marcar el checkbox `[x]` en este archivo.

### Reglas no negociables de este roadmap

- **C-01 va primero y no se reordena.** El flujo de impresión de punta a punta es el riesgo técnico #1 y todo lo demás depende de él.
- **Ningún change puede requerir la impresora real** para verificarse en CI ni en desarrollo. La validación usa golden tests del layout, `FileComandaPrinter` y una impresora local con puerto `FILE:` (silenciosa).
- **`Microsoft Print to PDF` no valida silencio**: siempre abre diálogo. Solo sirve para inspección visual de la comanda.
- **Beta = funciones 1 a 9 del brief** (US-001 a US-009). US-010 y US-011 son la **segunda iteración de la beta**. La exportación a Excel (C-16) es **post-beta** y no bloquea ningún change.
- **Gobernanza**: impresión y copia de seguridad = **ALTO** (tocan hardware y archivos del usuario); pedidos, menú y datos = **MEDIO**; UI y estilos = **BAJO**.
- **Stack fijo**: C# + .NET 10 LTS + WPF (MVVM) + SQLite (`Microsoft.Data.Sqlite`) + xUnit. Windows 10 x64, un solo usuario, sin login, 100 % offline.
- **Convención visual obligatoria** desde la primera pantalla: un único `src/VargasPizza.Desktop/Resources/Styles.xaml` importado en `App.xaml`, colores con contraste AA (≥ 4.5:1), tipografía grande para mostrador y foco de teclado visible. Prohibido usar colores, tamaños, márgenes, paddings o radios literales en las vistas; si falta un valor, se agrega al diccionario.
- **No se inventan funcionalidades fuera de la KB.** Toda ambigüedad se registra como pregunta abierta o dependencia dentro del change, sin decidirla.

---

## Árbol de dependencias

> Las fases son semánticas. El orden real de ejecución lo mandan los GATES de la sección siguiente.

```
C-01 impresion-punta-a-punta                 ← riesgo #1, SIEMPRE primero
  │
  ├── C-02 impresion-respaldo-escpos
  │
  └── C-03 modelo-de-datos
        │
        ├── C-10 copia-de-seguridad ────────────────────────────┐
        │                                                       │
        ├── C-04 editor-de-menu ─────────┐                      │
        │                                │                      │
        ├── C-05 clientes-por-telefono ──┤                      │
        │                                │                      │
        │                                └── C-06 carga-de-pedidos
        │                                      └── C-07 confirmacion-y-comanda
        │                                            ├── C-08 historial-y-reimpresion
        │                                            │     └── C-09 anulacion-de-pedidos
        │                                            │           ├── C-12 auditoria-visual-y-teclado
        │                                            │           └── C-15 correcciones-con-snapshot
        │                                            ├── C-14 opciones-por-producto
        │                                            └── C-16 exportacion-excel      (post-beta)
        │
        └── C-11 instalador-y-release ◄── requiere C-07 + C-10
              └── C-13 verificacion-rendimiento ◄── requiere C-07 + C-11
```

Lectura rápida:

- **C-01** es la raíz técnica (solución, impresión, pantalla de configuración).
- **C-03** es la raíz de datos: sin tablas no hay menú, clientes, pedidos ni backup.
- **C-06 → C-07 → C-08 → C-09** es la cadena del núcleo operable.
- **C-10** (backup) sale de C-03 y corre en paralelo con el núcleo.
- **C-14 → C-15** son la segunda iteración de la beta y no bloquean la primera.
- **C-16** es post-beta y no bloquea nada.

### Paralelismo por fase

> Cada GATE es un punto de sincronización. Los changes de un mismo GATE pueden ejecutarse en paralelo. `[Agente X]` es una asignación sugerida, no obligatoria.

```
GATE 0: ninguna
  → C-01 impresion-punta-a-punta            [Agente A — solo]

GATE 1: C-01 ✓                              ← PRIMER FORK (2 paralelos)
  → C-02 impresion-respaldo-escpos          [Agente B]
  → C-03 modelo-de-datos                    [Agente A]

GATE 2: C-03 ✓                              ← FORK (3 paralelos)
  → C-04 editor-de-menu                     [Agente C]
  → C-05 clientes-por-telefono              [Agente A]
  → C-10 copia-de-seguridad                 [Agente B]

GATE 3: C-04 ✓ + C-05 ✓
  → C-06 carga-de-pedidos                   [Agente A]

GATE 4: C-06 ✓
  → C-07 confirmacion-y-comanda             [Agente A]

GATE 5: C-07 ✓                              ← FORK
  → C-08 historial-y-reimpresion            [Agente C]
  → C-11 instalador-y-release               [Agente B — si C-10 ✓]
  → C-14 opciones-por-producto              [Agente A — segunda iteración]

GATE 6: C-08 ✓
  → C-09 anulacion-de-pedidos               [Agente C]
  → C-16 exportacion-excel                  [Agente B — post-beta, no bloquea]

GATE 7: C-09 ✓
  → C-12 auditoria-visual-y-teclado         [Agente C]
  → C-15 correcciones-con-snapshot          [Agente A — segunda iteración]

GATE 8: C-11 ✓ + C-07 ✓
  → C-13 verificacion-rendimiento           [Agente B]
```

### Camino crítico (7 changes — mínimo irreducible)

```
C-01 → C-03 → C-04 → C-06 → C-07 → C-08 → C-09
```

- **C-05** corre en paralelo con C-04 y también es prerequisito de C-06: la cadena no se alarga, pero C-06 no arranca hasta que ambas estén.
- **C-11** (instalador) cierra la entrega en paralelo y no alarga el camino.
- **C-02** (respaldo ESC/POS), **C-10** (backup), **C-12** y **C-13** son ramas paralelas de confiabilidad y calidad; no son irreducibles para que la beta funcione, pero sí para entregarla con la mitigación completa del riesgo de impresión.

### Plan óptimo con 3 agentes

```
Paso │ Agente A (Núcleo y dominio)      │ Agente B (Datos y entrega)   │ Agente C (UI y pantallas)
─────┼──────────────────────────────────┼──────────────────────────────┼───────────────────────────
  1  │ C-01 impresion-punta-a-punta     │ —                            │ —
  2  │ C-03 modelo-de-datos             │ C-02 impresion-respaldo      │ —
  3  │ C-05 clientes-por-telefono       │ C-10 copia-de-seguridad      │ C-04 editor-de-menu
  4  │ C-06 carga-de-pedidos            │ —                            │ —
  5  │ C-07 confirmacion-y-comanda      │ —                            │ —
  6  │ C-14 opciones-por-producto       │ C-11 instalador-y-release    │ C-08 historial-y-reimpresion
  7  │ —                                │ C-16 exportacion-excel       │ C-09 anulacion-de-pedidos
  8  │ C-15 correcciones-con-snapshot   │ C-13 verificacion-rendimiento│ C-12 auditoria-visual-y-teclado
```

---

## FASE 0 — Impresión de punta a punta (riesgo #1)

> Esta fase existe para responder una sola pregunta antes de invertir en el resto: **¿la app imprime sin diálogo en la impresora elegida?** Todo lo demás del roadmap asume que la respuesta es sí.

### [C-01] `impresion-punta-a-punta`

- **Estado**: `[ ]` pendiente
- **Scope**: Flujo completo "comanda de prueba → impresora elegida en configuración → Imprimir prueba → impresión silenciosa", con la base mínima de la solución.
  - Solución `VargasPizza.sln` con `src/VargasPizza.Desktop` (WPF, `net10.0-windows`), `src/VargasPizza.Core`, `src/VargasPizza.Data`, `src/VargasPizza.Printing` y `tests/VargasPizza.{Core,Data,Printing}.Tests`; `Directory.Build.props` con nullable e `ImplicitUsings` habilitados.
  - Shell mínimo de la app: `App.xaml` importando `src/VargasPizza.Desktop/Resources/Styles.xaml` como **único** `ResourceDictionary`; `MainWindow` con navegación a **Configuración → Impresión**. El diccionario define colores con contraste AA (≥ 4.5:1), tipografía grande para mostrador y estilo de foco de teclado visible; ninguna vista usa colores, tamaños, márgenes ni radios literales.
  - Contrato `ComandaDocument` (número, fecha/hora, nombre, teléfono, domicilio, ítems con código/nombre/precio/cantidad/nota, total y forma de pago) y `ComandaLayout` (líneas, columnas, wrapping, fuente).
  - `ComandaLayoutEngine` determinístico: wrapping por ancho de papel, alineación de precios, bloque de total y tamaño de letra; no conoce ninguna impresora.
  - `IComandaPrinter` + `FileComandaPrinter` (escribe el layout renderizado a un archivo, para desarrollo y tests) + `WindowsDriverComandaPrinter` (`System.Drawing.Printing.PrintDocument` con `PrinterSettings.PrinterName`, sin diálogo; `PrintController` inyectable para tests).
  - Pantalla Configuración → Impresión: enumerar `PrinterSettings.InstalledPrinters`, seleccionar cola, ancho de papel (default 80 mm) y tamaño de letra (default 9 pt); botón **Imprimir prueba** con un `ComandaDocument` de ejemplo; mostrar el error de Windows sin cerrar la pantalla; guardar en `configuracion`.
  - Migración 001: `schema_migrations` y `configuracion` (solo claves `impresion.*`). El resultado de **Imprimir prueba** se muestra en pantalla y se escribe en el log de desarrollo a través de `IImpresionLog`; la persistencia en `pedido_impresiones` llega con el esquema de dominio (C-03) y se conecta al camino de impresión en C-07.
  - Composición/inyección de dependencias mínima y log de desarrollo en `%LOCALAPPDATA%\Vargas Pizza\logs\`.
  - Tests: golden del layout (80 mm, dos tamaños de letra, nombres largos, acentos, total); integración que verifica que el camino driver invoca `StartDoc`/`PrintPage` y **nunca** abre diálogo; `FileComandaPrinter` recorre comanda completa; ViewModel de configuración con impresora inexistente y spooler caído.
  - CI: `.github/workflows/ci.yml` con `dotnet restore`, `dotnet build -c Release` y `dotnet test` en `windows-latest`, sin hardware.
  - Validación manual documentada: instalar una impresora local con puerto `FILE:` y ejecutar **Imprimir prueba**; dejar asentado que `Microsoft Print to PDF` solo sirve para inspección visual y no prueba silencio.
  - **Fuera de alcance**: camino ESC/POS (C-02), pedidos reales y persistencia de dominio (C-03 en adelante).
- **Dependencias**: ninguna
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/02_descripcion_general.md` §Stack tecnológico, §Impresión — caminos y contrato
  - `knowledge-base/06_funcionalidades.md` §US-001 (US-001 se completa en C-02 con el selector de modo)
  - `knowledge-base/07_flujos_principales.md` §Flujo 6: Configurar impresora e imprimir prueba
  - `knowledge-base/08_arquitectura_propuesta.md` §Estructura de directorios, §Impresión — diseño en detalle, §Estrategia de pruebas, §CI/CD
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-02, §DD-11

---

### [C-02] `impresion-respaldo-escpos`

- **Estado**: `[ ]` pendiente
- **Scope**: Camino de respaldo ESC/POS crudo detrás de `IComandaPrinter`, activable desde configuración.
  - `EscPosRawComandaPrinter`: serializa el mismo `ComandaLayout` a comandos ESC/POS (inicialización, codepage y acentos, énfasis, ancho, corte) y lo envía con `WritePrinter` sobre un handle `RAW` de la cola elegida, sin diálogo.
  - Selector `impresion.modo = driver | escpos` en Configuración → Impresión; **Imprimir prueba** usa el camino seleccionado y registra el intento con `camino` correcto.
  - Mensaje de diagnóstico cuando falla: mostrar el error del spooler y sugerir probar el otro camino (RN-CON-03).
  - Golden de bytes de ESC/POS: secuencia de inicialización, wrapping a 80 mm, acentos, énfasis y corte; test que verifica que este camino no usa GDI.
  - Manejo de `PrintResult` con éxito/error y registro del intento a través de `IImpresionLog` (implementación de archivo en desarrollo; la persistencia SQLite se conecta en C-07); ningún fallo debe tumbar la app.
  - Validación sin hardware: golden de bytes + impresora local con puerto `FILE:` o driver *Generic / Text Only*; no se requiere la Ocom.
  - **Pregunta abierta que no se decide acá**: si la Ocom OCPP-812 acepta ESC/POS crudo se valida en el local con **Imprimir prueba**; el camino es respaldo, no requisito para la entrega.
- **Dependencias**: C-01
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/02_descripcion_general.md` §Impresión — caminos y contrato
  - `knowledge-base/06_funcionalidades.md` §US-001
  - `knowledge-base/08_arquitectura_propuesta.md` §Impresión — diseño en detalle, §Estrategia de pruebas
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-02, §DD-11
  - `knowledge-base/10_preguntas_abiertas.md` §IN-03, §Preguntas abiertas (Ocom/ESC-POS)

---

## FASE 1 — Modelo de datos

### [C-03] `modelo-de-datos`

- **Estado**: `[ ]` pendiente
- **Scope**: Esquema completo del dominio, migraciones, repositorios y servicios de datos sobre SQLite.
  - Migración 002: `productos`, `presentaciones` (código de exactamente 4 dígitos, `UNIQUE` global, precio en centavos, `activo`, `orden`), `clientes` (`telefono_normalizado` `UNIQUE`), `pedidos` (`fecha_operativa`, `numero_dia` con `UNIQUE(fecha_operativa, numero_dia)`, tipo, estado, total, anulación), `pedido_items` (snapshots `codigo_snapshot`, `nombre_snapshot`, `precio_unitario_centavos`, `cantidad`, `subtotal_centavos`, `nota`, `orden`), `pedido_impresiones` (`pedido_id` nullable, motivo `inicial|reimpresion|prueba`, camino, éxito, error) y `contador_comandas`.
  - Índices y `CHECK` del modelo: dinero `INTEGER` en centavos, fechas ISO-8601 en hora local, booleanos 0/1, `estado IN ('confirmado','anulado')`, `tipo IN ('mostrador','domicilio')`, `forma_pago IN ('efectivo','transferencia')`.
  - Runner de migraciones idempotente y versionado en `schema_migrations`; seed de `configuracion` con `operacion.hora_corte = 06:00`, `backup.carpeta`, `backup.retencion_dias = 30` y `app.ultima_copia_apertura_fecha`; **ningún producto** precargado.
  - Entidades y repositorios: `ProductoRepository`, `PresentacionRepository`, `ClienteRepository`, `PedidoRepository`, `ConfiguracionRepository`, `ContadorComandasRepository`; abstracción `IClock`.
  - `DiaOperativoService`: `fecha_operativa = date(fecha_hora_local - hora_de_corte)`; tests de borde 05:59 / 06:01 y de que cambiar la hora de corte no renumera pedidos existentes.
  - Reserva de número con `BEGIN IMMEDIATE` + `UNIQUE(fecha_operativa, numero_dia)`; tests de dos reservas simultáneas y de que un pedido anulado **no** libera su número.
  - Tests de integración con SQLite en archivo temporal: migración desde cero, `UNIQUE` de código (incluida presentación inactiva), `UNIQUE` de teléfono, snapshots inmutables al cambiar el menú y constraints.
  - **No se crean todavía** `producto_opciones`, `opcion_valores` ni `pedido_correcciones`: llegan en C-14 y C-15 para no implementar la segunda iteración antes de tiempo.
  - **Preguntas abiertas que no se deciden acá**: tamaños de pizza, pizzas mitad y mitad y variedades de la empanada surtida. El esquema soporta N presentaciones por producto sin cambios; si el cliente pide componentes por ítem, se registra como pregunta abierta del change y no se inventa el modelo.
- **Dependencias**: C-01
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §Convenciones, §ERD, §Entidades, §Día operativo
  - `knowledge-base/05_reglas_de_negocio.md` §RN-MEN, §RN-PED, §RN-COM, §RN-CLI
  - `knowledge-base/08_arquitectura_propuesta.md` §Patrones aplicados (Repository, migraciones versionadas, snapshot)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-03, §DD-04, §DD-05, §DD-06
  - `knowledge-base/10_preguntas_abiertas.md` §Bloqueos por fase

---

## FASE 2 — Catálogo y clientes

> C-04 y C-05 pueden ejecutarse en paralelo después de C-03. Ambos son prerequisito de C-06.

### [C-04] `editor-de-menu`

- **Estado**: `[ ]` pendiente
- **Scope**: Editor de menú completo (US-002) con productos y presentaciones agrupadas.
  - Pantalla **Menú**: listado agrupado producto → presentaciones, con nombre, categoría opcional, código, precio y estado.
  - Alta y edición de producto (nombre, categoría, orden) y de presentación (nombre, código, precio, orden) bajo un producto padre existente.
  - Validación de código: exactamente 4 dígitos, solo números, `UNIQUE` global incluyendo presentaciones inactivas; mensaje que indica qué producto/presentación ya usa el código.
  - Precio en centavos ≥ 0; baja lógica (`activo = 0`) sin borrado físico y sin romper pedidos históricos.
  - ViewModels con tests de RN-MEN-01 a RN-MEN-06; test de integración de que editar un precio no modifica `pedido_items` existentes.
  - UI 100 % desde `Resources/Styles.xaml`: foco visible en grillas y formularios, tipografía grande, sin valores sueltos.
  - **Fuera de alcance**: opciones por producto (C-14).
- **Dependencias**: C-03
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §productos, §presentaciones
  - `knowledge-base/05_reglas_de_negocio.md` §Dominio: Menú (RN-MEN-01 a RN-MEN-06)
  - `knowledge-base/06_funcionalidades.md` §US-002
  - `knowledge-base/07_flujos_principales.md` §Flujo 4: Editar el menú
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-06

---

### [C-05] `clientes-por-telefono`

- **Estado**: `[ ]` pendiente
- **Scope**: Identificación de clientes por teléfono (US-006), lista para autocompletar en la carga de pedidos.
  - `TelefonoNormalizador` según RN-CLI-01: quita espacios, guiones, paréntesis y `+`; descarta prefijo `54`; aplica la regla de `9`/`15` de móvil argentino. Determinista y con tabla de casos.
  - `ClienteRepository`: buscar por `telefono_normalizado`, crear o actualizar cliente al confirmar un pedido, y resolver dos escrituras distintas del mismo número al mismo registro.
  - Actualización de `ultimo_domicilio` solo cuando el pedido es de tipo `domicilio`; un pedido de mostrador no borra el domicilio guardado (RN-CLI-03).
  - Autocompletado reutilizable (ViewModel/behavior) de nombre y último domicilio, con campos que siguen siendo editables.
  - Tests: normalización, `UNIQUE`, autocompletado, no borrado del domicilio en mostrador y actualización en domicilio.
  - **No se crea una pantalla de clientes**: la KB no la define; la gestión es implícita desde los pedidos.
- **Dependencias**: C-03
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §clientes
  - `knowledge-base/05_reglas_de_negocio.md` §Dominio: Clientes (RN-CLI-01 a RN-CLI-03), §RN-PED-09, §RN-PED-10
  - `knowledge-base/06_funcionalidades.md` §US-006
  - `knowledge-base/07_flujos_principales.md` §Flujo 1: Cargar y confirmar un pedido con impresión
  - `knowledge-base/09_decisiones_y_supuestos.md` §SU-01

---

## FASE 3 — Núcleo operable: carga, comanda, historial y anulación

> Esta fase reemplaza la comanda de papel. Al terminar C-09, la beta núcleo está completa y usable.

### [C-06] `carga-de-pedidos`

- **Estado**: `[ ]` pendiente
- **Scope**: Pantalla de carga de pedido teclado-primero (US-003) con total automático (US-004).
  - Foco inicial en el campo código al abrir la pantalla; 4 dígitos + Enter agrega el ítem con cantidad 1 usando la presentación activa.
  - Código inexistente o inactivo: error claro en pantalla, sin agregar nada.
  - Edición de cantidad, nota libre por ítem (sale impresa) y quitar ítem antes de confirmar.
  - Cabecera: nombre, teléfono (con autocompletado de C-05), tipo mostrador/domicilio, dirección, horario estimado precargado en "ahora + 40 min" y editable, forma de pago obligatoria.
  - Total recalculado en cada alta, baja o cambio de cantidad, con objetivo < 150 ms; suma de subtotales sin envío, descuentos ni recargos.
  - Validaciones al confirmar: total > 0, dirección obligatoria si es domicilio, forma de pago obligatoria; mensajes que indiquen el campo.
  - Navegación Tab / Shift+Tab y atajo de confirmación; recorrido completo de un pedido de 4 ítems sin mouse.
  - ViewModel con tests del flujo código → Enter → ítem y total, teclado, validaciones y estabilidad de la medición de alta de ítem.
  - UI 100 % desde `Resources/Styles.xaml`, con foco de teclado visible en cada control.
  - **Fuera de alcance**: persistencia y numeración (C-07), opciones estructuradas (C-14), recuperación de borrador tras corte de luz (pregunta abierta; no se implementa).
- **Dependencias**: C-03, C-04, C-05
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §pedidos, §pedido_items
  - `knowledge-base/05_reglas_de_negocio.md` §RN-PED-01 a RN-PED-09, §RN-MEN-05
  - `knowledge-base/06_funcionalidades.md` §US-003, §US-004
  - `knowledge-base/07_flujos_principales.md` §Flujo 1: Cargar y confirmar un pedido con impresión
  - `knowledge-base/08_arquitectura_propuesta.md` §Patrones aplicados (MVVM), §Estrategia de pruebas (ViewModel)

---

### [C-07] `confirmacion-y-comanda`

- **Estado**: `[ ]` pendiente
- **Scope**: Confirmación transaccional del pedido y emisión automática de la comanda (US-005).
  - Caso de uso `ConfirmarPedido`: valida reglas, abre `BEGIN IMMEDIATE`, reserva el número de `contador_comandas` para el día operativo, inserta `pedidos` + `pedido_items` con snapshots y total, y hace commit.
  - Construcción de `ComandaDocument` desde los datos guardados (no desde el menú vivo) y layout con ancho y tamaño de letra configurados.
  - Impresión silenciosa por `IComandaPrinter` según `impresion.modo`; nunca abre diálogo.
  - Conexión de `IImpresionLog` a la tabla `pedido_impresiones` (creada en C-03) y registro de cada intento con motivo `inicial`, camino, éxito y error; **Imprimir prueba** pasa a persistir con motivo `prueba`.
  - Fallo de impresión: el pedido queda guardado en estado confirmado, se muestra el error y se ofrece **Reimprimir**; no se hace rollback del pedido (RN-COM-05).
  - Sin impresora configurada: aviso previo y opción explícita de guardar sin imprimir.
  - Configuración → Operación: hora de corte editable (RN-CON-01) con aviso de que no renumera pedidos existentes (RN-CON-02).
  - Tests: correlativo por día operativo y reinicio a 1; dos confirmaciones simultáneas no repiten número; anulado no libera número; snapshot inmutable al cambiar el menú; comanda contiene número, hora, nombre, teléfono, domicilio si aplica, código + producto, precio por ítem, total y forma de pago; fallo de impresión no pierde el pedido; driver no abre diálogo con `PrintController` falso; recorrido E2E con `FileComandaPrinter`.
- **Dependencias**: C-01, C-02, C-06
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/02_descripcion_general.md` §Impresión — caminos y contrato
  - `knowledge-base/04_modelo_de_datos.md` §pedidos, §pedido_items, §pedido_impresiones, §contador_comandas
  - `knowledge-base/05_reglas_de_negocio.md` §RN-COM-01 a RN-COM-10, §RN-PED-03, §RN-CON-01, §RN-CON-02
  - `knowledge-base/06_funcionalidades.md` §US-005
  - `knowledge-base/07_flujos_principales.md` §Flujo 1: Cargar y confirmar un pedido con impresión

---

### [C-08] `historial-y-reimpresion`

- **Estado**: `[ ]` pendiente
- **Scope**: Historial de pedidos y reimpresión desde snapshots (US-007).
  - Pantalla **Historial**: por defecto el día operativo actual; columnas número, hora, nombre, total y estado.
  - Cambio de fecha y búsqueda por número o teléfono.
  - Detalle del pedido con ítems, cantidades, precios, nota, total y forma de pago.
  - **Reimprimir** reconstruye la comanda desde `pedido_items` congelados, no desde el menú actual; registra el intento con motivo `reimpresion` en `pedido_impresiones`.
  - Pedido anulado: la reimpresión se marca visualmente como **ANULADO** (RN-ANU-02).
  - ViewModel tests de filtros por fecha, búsqueda y reimpresión; integración con `FileComandaPrinter`.
  - UI desde `Resources/Styles.xaml`, con foco visible y navegación por teclado en la grilla.
- **Dependencias**: C-07
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §pedidos, §pedido_items, §pedido_impresiones
  - `knowledge-base/05_reglas_de_negocio.md` §RN-COM-06, §RN-COM-07, §RN-ANU-02
  - `knowledge-base/06_funcionalidades.md` §US-007
  - `knowledge-base/07_flujos_principales.md` §Flujo 2: Reimprimir una comanda
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-04

---

### [C-09] `anulacion-de-pedidos`

- **Estado**: `[ ]` pendiente
- **Scope**: Anulación con trazabilidad, sin borrado (US-008).
  - Acción **Anular** en el detalle del historial, con confirmación y motivo opcional.
  - Actualización de `estado = 'anulado'`, `anulado_at` y `anulado_motivo`; la fila nunca se elimina.
  - Validación de que un pedido ya anulado no se modifica y no se puede reactivar (RN-ANU-03).
  - El historial refleja el estado sin ocultar el pedido; la reimpresión sigue marcada como **ANULADO**.
  - Tests de anulación, doble anulación, persistencia del motivo y de que el número de comanda no se libera.
  - UI desde `Resources/Styles.xaml`, con foco visible en el diálogo de confirmación.
- **Dependencias**: C-08
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §pedidos
  - `knowledge-base/05_reglas_de_negocio.md` §Dominio: Anulación y corrección (RN-ANU-01 a RN-ANU-03), §RN-COM-02
  - `knowledge-base/06_funcionalidades.md` §US-008
  - `knowledge-base/07_flujos_principales.md` §Flujo 3: Anular un pedido
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-05

---

## FASE 4 — Confiabilidad y entrega

### [C-10] `copia-de-seguridad`

- **Estado**: `[ ]` pendiente
- **Scope**: Copia de seguridad local con tres disparadores y retención (US-009).
  - Servicio de backup con el mecanismo online de SQLite (o copia con la base cerrada), nunca copiando un archivo abierto a mano (RN-SEG-07).
  - Disparadores: al cerrar la app; en la primera apertura de cada día operativo usando `app.ultima_copia_apertura_fecha`; y botón manual **Copiar ahora**.
  - Carpeta destino configurable con default `%USERPROFILE%\Documents\Vargas Pizza\Copia de seguridad`; retención de 30 días aplicada **solo después** de una copia exitosa.
  - Registro en `backup_log` con motivo (`cierre`, `apertura`, `manual`), ruta, éxito, error y tamaño.
  - Un fallo (carpeta sin permisos, disco lleno) se registra, se avisa y **no** bloquea el cierre ni borra copias anteriores (RN-SEG-06).
  - Pantalla Configuración → Copia de seguridad: carpeta, botón manual y último resultado.
  - Tests con directorios temporales: copia válida y legible, apertura una sola vez por fecha operativa, retención, fallo simulado sin borrado y cierre no bloqueante.
  - **Fuera de alcance**: restauración automática. La KB no la define como funcionalidad; si el cliente la pide, se registra como pregunta abierta y change aparte.
- **Dependencias**: C-03
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §configuracion, §backup_log, §Día operativo
  - `knowledge-base/05_reglas_de_negocio.md` §Dominio: Copia de seguridad (RN-SEG-01 a RN-SEG-07)
  - `knowledge-base/06_funcionalidades.md` §US-009
  - `knowledge-base/07_flujos_principales.md` §Flujo 5: Copia de seguridad
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-07

---

### [C-11] `instalador-y-release`

- **Estado**: `[ ]` pendiente
- **Scope**: Paquete instalable y publicación automática al crear un tag.
  - `dotnet publish -c Release -r win-x64 --self-contained` de `VargasPizza.Desktop`.
  - Script de Inno Setup 6: instalación por usuario, accesos directos, datos en `%LOCALAPPDATA%\Vargas Pizza\`, desinstalación sin borrar la base, sin firma de código.
  - `.github/workflows/release.yml`: en tag `v*`, publish + compilar Inno Setup + subir el `.exe` a GitHub Releases.
  - `.github/workflows/ci.yml` definitivo: restore, build Release y tests en cada push/PR, sin hardware.
  - README de entrega con el paso a paso de SmartScreen ("Windows protegió tu PC") y la aclaración de que las actualizaciones son manuales.
  - Verificación de que ni el build ni el instalador requieren una impresora instalada.
- **Dependencias**: C-07, C-10
- **Governance**: ALTO
- **Leer antes**:
  - `knowledge-base/01_vision_y_objetivos.md` §Alcance v0.1 (beta), §Fuera de alcance
  - `knowledge-base/02_descripcion_general.md` §Stack tecnológico
  - `knowledge-base/08_arquitectura_propuesta.md` §Estructura de directorios, §Seguridad (datos en `%LOCALAPPDATA%`), §CI/CD
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-01 (trade-offs del stack)

---

### [C-12] `auditoria-visual-y-teclado`

- **Estado**: `[ ]` pendiente
- **Scope**: Auditoría de la convención visual obligatoria y del recorrido 100 % teclado, sin sistema de diseño formal (`visual_design.mode = off`).
  - Verificar que existe un único `src/VargasPizza.Desktop/Resources/Styles.xaml`, importado en `App.xaml`, y que **ninguna** vista define colores, tamaños de fuente, márgenes, paddings o radios literales.
  - Documentar y verificar contraste AA (≥ 4.5:1) de texto y fondo con los recursos con nombre del diccionario.
  - Verificar tipografía grande para lectura de mostrador y estilo de foco de alto contraste en **todo** control enfocable.
  - Recorrido manual y con tests de ViewModel del flujo de carga de pedido: foco inicial en código, Tab / Shift+Tab, atajos de confirmar e imprimir, foco visible en cada pantalla.
  - Entregable: correcciones centralizadas en `Styles.xaml` (no en las vistas) y checklist firmado por pantalla.
  - **No se crean mockups ni tokens nuevos**: la política visual del proyecto es `off` y esta convención no es un sistema de diseño.
- **Dependencias**: C-04, C-06, C-07, C-08, C-09
- **Governance**: BAJO
- **Leer antes**:
  - `design-system/README.md` §Convención mínima obligatoria (no es un sistema de diseño)
  - `knowledge-base/06_funcionalidades.md` §US-003 (carga teclado primero)
  - `knowledge-base/08_arquitectura_propuesta.md` §Estructura de directorios (`Desktop/Resources`), §Patrones aplicados (MVVM)
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-09

---

### [C-13] `verificacion-rendimiento`

- **Estado**: `[ ]` pendiente
- **Scope**: Protocolo y medición de las metas no funcionales, sin cambiar funcionalidades.
  - Arranque en frío ≤ 3 s: primera ejecución tras reiniciar Windows hasta que la pantalla acepta el primer código; mediana de 3.
  - RAM ≤ 150 MB: suma del Working Set **privado** de todos los procesos de la app, con un pedido cargado y 10 minutos de uso normal, en la PC del local o una equivalente.
  - Alta de ítem < 150 ms: desde Enter hasta ítem visible y total actualizado; mediana de 20.
  - Impresión < 2 s: desde confirmar hasta que Windows reporta el trabajo entregado al spooler, con una cola local (incluida una con puerto `FILE:` para ejercitar el spooler); sin Ocom ni `FileComandaPrinter` para esta medición.
  - Instrumentación ligera (stopwatch + log de desarrollo) y documento de resultados con fecha, equipo y condiciones.
  - Si una meta no se cumple, se documenta la desviación y se registra como pregunta abierta/riesgo; no se optimiza a ciegas ni se cambia el alcance.
- **Dependencias**: C-07, C-10, C-11
- **Governance**: BAJO
- **Leer antes**:
  - `knowledge-base/01_vision_y_objetivos.md` §Métricas de éxito
  - `knowledge-base/02_descripcion_general.md` §Arquitectura general (monoproceso WPF)
  - `knowledge-base/08_arquitectura_propuesta.md` §Estrategia de pruebas
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-01, §DD-09

---

## FASE 5 — Beta, segunda iteración

> Se construye después de que el núcleo (C-01 a C-09) esté archivado. No bloquea la primera beta usable.

### [C-14] `opciones-por-producto`

- **Estado**: `[ ]` pendiente
- **Scope**: Opciones configurables por producto (US-010), con snapshot e impresión.
  - Migración 003: `producto_opciones` (`tipo single|multi`, `requerido`, `activo`, `orden`) y `opcion_valores` (`activo`, `orden`), **sin precio adicional** (DD-08).
  - Editor de menú: alta, edición y baja lógica de opciones y valores por producto.
  - Carga de pedido: al agregar un ítem se ofrecen solo las opciones de ese producto; las requeridas impiden agregar sin elegir; selección por teclado.
  - Snapshot en `pedido_item_opciones` (`opcion_nombre`, `valor_nombre`) al confirmar el pedido.
  - Impresión de las opciones elegidas en la comanda, dentro del bloque del ítem.
  - Tests: opciones por producto, requeridas, snapshot inmutable, impresión y total sin cambios por opciones.
  - **Dependencia de negocio**: la KB marca las opciones estructuradas como propuesta (IN-01). Confirmar con el cliente antes de construir; si prefiere solo nota libre, el change no se ejecuta y la beta sigue con la nota libre de C-06.
- **Dependencias**: C-04, C-06, C-07
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §producto_opciones, §opcion_valores, §pedido_item_opciones
  - `knowledge-base/05_reglas_de_negocio.md` §RN-MEN-07
  - `knowledge-base/06_funcionalidades.md` §US-010
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-08
  - `knowledge-base/10_preguntas_abiertas.md` §IN-01

---

### [C-15] `correcciones-con-snapshot`

- **Estado**: `[ ]` pendiente
- **Scope**: Corrección de un pedido con snapshot antes/después y reimpresión automática (US-011).
  - Migración 004: `pedido_correcciones` (`fecha_hora`, `motivo`, `snapshot_antes` JSON, `snapshot_despues` JSON, `reimprimio`).
  - Flujo **Corregir** desde el detalle del historial: reutiliza el formulario de carga (C-06) en modo corrección sobre el pedido guardado; los ítems nuevos o reemplazados congelan el precio vigente (RN-PED-03), los ítems no tocados conservan su snapshot, y el total se recalcula y se guarda.
  - Guardar snapshot antes/después y motivo; reimprimir automáticamente después de guardar y registrar el intento.
  - El historial indica que el pedido fue corregido; un pedido anulado no se corrige (RN-ANU-03).
  - Tests: snapshot completo, recálculo de total, reimpresión automática, pedido anulado rechazado y persistencia del motivo.
  - **Pregunta abierta que se registra en el change**: la KB no detalla qué campos exactos son corregibles. La propuesta parte de reutilizar el formulario de carga; cualquier recorte adicional se valida con el cliente antes de implementar, no se decide en código.
- **Dependencias**: C-07, C-08, C-09
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/04_modelo_de_datos.md` §pedido_correcciones, §pedido_items
  - `knowledge-base/05_reglas_de_negocio.md` §RN-ANU-04, §RN-PED-01, §RN-PED-03
  - `knowledge-base/06_funcionalidades.md` §US-011
  - `knowledge-base/09_decisiones_y_supuestos.md` §DD-04, §DD-05

---

## FASE 6 — Post-beta

> Nada de esta fase bloquea la beta. Puede ejecutarse después de entregar C-01 a C-13.

### [C-16] `exportacion-excel`

- **Estado**: `[ ]` pendiente
- **Scope**: Exportación del historial a Excel (US-012). **Explícitamente fuera de la beta.**
  - Exportar pedidos e ítems de un rango de fechas/días operativos a un archivo `.xlsx`, respetando snapshots de código, nombre, precio, cantidad, subtotal, total y estado.
  - Acción manual desde el historial; sin escritura automática ni integración con la nube.
  - Tests de exportación con pedidos confirmados, anulados y corregidos; verificar que no altera la base.
  - **Decisión abierta del change**: la librería de Excel no está definida en la KB ni en el stack fijo. Se evalúa en la propuesta (por ejemplo OpenXML/ClosedXML) sin agregar dependencias a la beta.
  - **No modifica** los flujos de C-01 a C-15 ni el esquema de datos.
- **Dependencias**: C-08
- **Governance**: MEDIO
- **Leer antes**:
  - `knowledge-base/01_vision_y_objetivos.md` §Fuera de alcance
  - `knowledge-base/04_modelo_de_datos.md` §pedidos, §pedido_items
  - `knowledge-base/06_funcionalidades.md` §US-012
  - `knowledge-base/10_preguntas_abiertas.md` §Preguntas abiertas (prioridad baja)

---

## Resumen de changes

| ID | Change | Fase | Release | Governance | Dependencias |
|---|---|---|---|---|---|
| C-01 | `impresion-punta-a-punta` | 0 — Impresión | Beta (1-9) | ALTO | — |
| C-02 | `impresion-respaldo-escpos` | 0 — Impresión | Beta (1-9) | ALTO | C-01 |
| C-03 | `modelo-de-datos` | 1 — Datos | Beta (1-9) | MEDIO | C-01 |
| C-04 | `editor-de-menu` | 2 — Catálogo | Beta (1-9) | MEDIO | C-03 |
| C-05 | `clientes-por-telefono` | 2 — Catálogo | Beta (1-9) | MEDIO | C-03 |
| C-06 | `carga-de-pedidos` | 3 — Núcleo | Beta (1-9) | MEDIO | C-03, C-04, C-05 |
| C-07 | `confirmacion-y-comanda` | 3 — Núcleo | Beta (1-9) | ALTO | C-01, C-02, C-06 |
| C-08 | `historial-y-reimpresion` | 3 — Núcleo | Beta (1-9) | MEDIO | C-07 |
| C-09 | `anulacion-de-pedidos` | 3 — Núcleo | Beta (1-9) | MEDIO | C-08 |
| C-10 | `copia-de-seguridad` | 4 — Confiabilidad | Beta (1-9) | ALTO | C-03 |
| C-11 | `instalador-y-release` | 4 — Entrega | Beta (1-9) | ALTO | C-07, C-10 |
| C-12 | `auditoria-visual-y-teclado` | 4 — Calidad | Beta (1-9) | BAJO | C-04, C-06, C-07, C-08, C-09 |
| C-13 | `verificacion-rendimiento` | 4 — Calidad | Beta (1-9) | BAJO | C-07, C-10, C-11 |
| C-14 | `opciones-por-producto` | 5 — Segunda iteración | Beta (US-010) | MEDIO | C-04, C-06, C-07 |
| C-15 | `correcciones-con-snapshot` | 5 — Segunda iteración | Beta (US-011) | MEDIO | C-07, C-08, C-09 |
| C-16 | `exportacion-excel` | 6 — Post-beta | **Post-beta** | MEDIO | C-08 |

**Total**: 16 changes en 7 fases.
**Beta núcleo (primera entrega usable)**: C-01 a C-13.
**Beta segunda iteración**: C-14 y C-15.
**Post-beta, sin bloquear nada**: C-16.

### Primer change recomendado

`C-01` — `impresion-punta-a-punta`.

Para arrancar:

```
/opsx:propose impresion-punta-a-punta
```

Antes de proponerlo, confirmar que `openspec/changes/archive/` está vacío (es el primer change del proyecto) y leer `knowledge-base/02_descripcion_general.md` §Impresión y `knowledge-base/08_arquitectura_propuesta.md` §Estrategia de pruebas.
