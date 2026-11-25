# 02 — Descripción General

## Stack tecnológico

| Capa | Tecnología | Versión mínima | Por qué |
|---|---|---|---|
| Lenguaje | C# | 13 (C# 13) | Un solo lenguaje para UI, dominio, datos e impresión. |
| Runtime / SDK | .NET | 10 LTS | Soporte hasta nov-2028. .NET 8 sale de soporte en nov-2026. |
| UI | WPF con MVVM | incluida en .NET 10 | Monoproceso, impresión madura, costo de toolchain bajo. |
| Base de datos | SQLite | 3.40+ | Local, embebida, transaccional, sin servidor. |
| Acceso a datos | Microsoft.Data.Sqlite + SQL explícito | 10.x | Sin ORM en la beta: menos magia, errores más claros. |
| Impresión | `System.Drawing.Printing` / `System.Printing` | incluida | Camino principal por driver de Windows. |
| Pruebas | xUnit | 2.x | Estándar de la comunidad .NET. |
| Instalador | Inno Setup | 6.x | `.exe` simple, sin firma, silencioso. |
| CI/CD | GitHub Actions (`windows-latest`) | — | Build + tests + release al tag. |

Sin dependencias en la nube, sin telemetría, sin APIs de terceros.

## Arquitectura general

```
┌──────────────────────────────────────────────────────────┐
│ VargasPizza.Desktop (WPF)                                │
│  Views (XAML) ←→ ViewModels ←→ Servicios de aplicación   │
└───────────────────────────┬──────────────────────────────┘
                            │ interfaces (Core)
┌───────────────────────────▼──────────────────────────────┐
│ VargasPizza.Core                                         │
│  Entidades · Reglas de negocio · Casos de uso            │
│  IPedidoRepository · IMenúRepository · IComandaPrinter   │
└──────────────┬────────────────────────────┬──────────────┘
               │                            │
┌──────────────▼──────────────┐  ┌──────────▼───────────────┐
│ VargasPizza.Data            │  │ VargasPizza.Printing     │
│  SQLite · migraciones       │  │  Layout de comanda       │
│  repositorios               │  │  Driver / ESC-POS / File │
└──────────────┬──────────────┘  └──────────┬───────────────┘
               │                            │
        ┌──────▼──────┐              ┌──────▼──────────────┐
        │  SQLite     │              │ Spooler de Windows  │
        │  (archivo)  │              │ (impresora elegida) │
        └─────────────┘              └─────────────────────┘
```

**Decisiones de alto nivel y su justificación**

1. **WPF en vez de un webview**: el requisito de ≤150 MB medido como suma privada de todos los procesos descarta Chromium/WebView2 (3–4 procesos). WPF corre en un solo proceso. Además, `System.Printing` resuelve la impresión silenciosa sin FFI ni plugins, que es el riesgo #1 del proyecto.
2. **Monolito de escritorio en capas, no microservicios**: un usuario, una PC, sin red. Cualquier separación de procesos suma RAM y complejidad sin beneficio.
3. **SQLite con SQL explícito**: el dominio es chico y estable; un ORM agrega peso y errores opacos para un principiante. Las migraciones son scripts versionados.
4. **Impresión detrás de una interfaz**: el camino principal usa el driver de Windows; el respaldo emite ESC/POS crudo. La app nunca habla con el hardware directamente.

## Integraciones externas

| Servicio | Propósito | Tipo | Notas |
|---|---|---|---|
| Spooler de impresión de Windows | Enviar la comanda a la impresora elegida | API local (`System.Printing` / Win32 `WritePrinter`) | Única integración. Imprime sin diálogo. |

No hay API REST, webhooks, pagos, mensajería ni servicios remotos.

## API / superficie expuesta

No aplica: es una app de escritorio monousuario. La "superficie" son las pantallas y los casos de uso internos.

## Impresión — caminos y contrato

```
ComandaDocument (datos congelados del pedido)
        │
        ▼
ComandaLayoutEngine  ──►  ComandaLayout (líneas, posiciones, wrapping, fuente)
        │
        ├─► WindowsDriverComandaPrinter   (PRINCIPAL: driver de Windows, GDI/XPS)
        ├─► EscPosRawComandaPrinter       (RESPALDO: bytes ESC/POS por WritePrinter)
        └─► FileComandaPrinter            (DESARROLLO/TESTS: escribe a archivo)
```

Todos implementan `IComandaPrinter`. La selección del camino vive en configuración (`impresion.modo = driver | escpos`), con `driver` por defecto y `escpos` como respaldo activable desde la pantalla de configuración si el driver rinde mal.

**Por qué el driver es el principal**: la impresora del cliente ya está instalada y en uso con su driver de Windows. Programar ESC/POS obligaría a conocer el modelo, que es exactamente lo que el brief prohíbe. El driver además respeta el tamaño de papel y maneja el corte si el driver lo soporta.

**Por qué existe el respaldo**: hay drivers térmicos que rasterizan mal o ignoran el ancho configurado. Si en el local la comanda sale cortada o ilegible, se cambia a ESC/POS sin tocar el resto del sistema. Como no se puede probar con la impresora real antes de la entrega, tener el segundo camino listo reduce el riesgo de la instalación.
