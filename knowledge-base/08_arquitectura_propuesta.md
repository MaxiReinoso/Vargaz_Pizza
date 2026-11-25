# 08 — Arquitectura Propuesta

## Patrones aplicados

| Patrón | Dónde se usa | Por qué |
|---|---|---|
| MVVM | `VargasPizza.Desktop` | Separa UI de lógica; permite testear el flujo teclado-primero sin abrir ventanas. |
| Capas (Core / Data / Printing / Desktop) | Toda la solución | El dominio no conoce SQLite ni impresoras. |
| Repository | `Core` define interfaces; `Data` las implementa | Tests de dominio sin base de datos. |
| Strategy / Adapter | `IComandaPrinter` con tres implementaciones | Camino principal por driver, respaldo ESC/POS y camino de archivo para desarrollo. |
| Result / errores explícitos | Casos de uso de `Core` | La impresión y el backup fallan de forma esperada; no se usan excepciones para control de flujo. |
| Migraciones versionadas | `VargasPizza.Data/Migrations` | Esquema reproducible desde cero y testeable. |
| Snapshot de datos | `pedido_items`, `pedido_item_opciones`, `pedido_correcciones` | El historial no depende del menú vivo. |

## Estructura de directorios

```
Vargaz_Pizza/
├── VargasPizza.sln
├── src/
│   ├── VargasPizza.Desktop/
│   │   ├── App.xaml / App.xaml.cs
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   ├── Controls/
│   │   ├── Resources/            # estilos y tokens visuales
│   │   └── Composition/          # inyección de dependencias
│   ├── VargasPizza.Core/
│   │   ├── Entities/
│   │   ├── UseCases/
│   │   ├── Rules/
│   │   ├── Abstractions/         # repositorios, IComandaPrinter, IClock
│   │   └── Services/             # día operativo, normalización de teléfono
│   ├── VargasPizza.Data/
│   │   ├── Migrations/
│   │   ├── Repositories/
│   │   └── Sqlite/
│   └── VargasPizza.Printing/
│       ├── ComandaDocument.cs
│       ├── ComandaLayoutEngine.cs
│       ├── WindowsDriverComandaPrinter.cs
│       ├── EscPosRawComandaPrinter.cs
│       └── FileComandaPrinter.cs
├── tests/
│   ├── VargasPizza.Core.Tests/
│   ├── VargasPizza.Data.Tests/
│   └── VargasPizza.Printing.Tests/
├── .github/workflows/
│   ├── ci.yml                     # build + tests en push
│   └── release.yml                # en tag v*: publish + Inno Setup + Release
├── docs/                          # brief del cliente
├── discovery/
├── knowledge-base/
├── design-system/
└── openspec/
```

## Impresión — diseño en detalle

```
ComandaDocument            datos congelados del pedido (número, hora, cliente, ítems, total)
      │
      ▼
ComandaLayoutEngine        calcula líneas, wrapping, alineación y posiciones
      │                    en función de ancho_papel_mm y tamano_letra_pt
      ▼
ComandaLayout              modelo determinístico, sin dependencia de impresora
      │
      ├── WindowsDriverComandaPrinter   (principal)
      ├── EscPosRawComandaPrinter       (respaldo)
      └── FileComandaPrinter            (desarrollo y tests)
```

**Camino principal — driver de Windows**
`System.Drawing.Printing.PrintDocument` con `PrinterSettings.PrinterName` configurado. Se dibuja el layout con `Graphics.DrawString`. No se instancia ningún diálogo. Ventaja: funciona con cualquier impresora instalada y con el driver que el cliente ya usa. Desventaja: el resultado final depende del driver, así que **no hay bytes propios que comparar**.

**Camino de respaldo — ESC/POS crudo**
El mismo `ComandaLayout` se serializa a comandos ESC/POS y se envía con `WritePrinter` sobre un handle `RAW`. Ventaja: control total y bytes determinísticos. Desventaja: requiere que la impresora hable ESC/POS y hay que cuidar la codificación de acentos.

**Camino de desarrollo — archivo**
Escribe el layout renderizado a un archivo de texto. Permite probar el flujo completo sin ninguna impresora.

**Regla de diseño**: ningún proyecto fuera de `VargasPizza.Printing` conoce `System.Drawing.Printing` ni Win32. `Desktop` solo usa `IComandaPrinter`.

## Estrategia de pruebas

Alineada con el camino de impresión elegido: **el driver no produce bytes comparables**, así que la verificación principal es del **layout**, no del flujo de bytes del dispositivo.

| Nivel | Qué se prueba | Cómo | Cuándo |
|---|---|---|---|
| **Unidad — dominio** | Totales, precio congelado, día operativo, correlativo, anular, normalización de teléfono, validación de códigos | xUnit puro, sin I/O | Cada cambio (TDD) |
| **Unidad — layout de comanda** | Orden de líneas, wrapping a 80 mm, tamaño de letra, alineación de precios, bloque de total, íconos de anulado | `ComandaLayoutEngine` → comparación contra archivos `.golden` del **layout** | Cada cambio de impresión |
| **Unidad — ESC/POS** | Secuencia de bytes del camino de respaldo (corte, énfasis, ancho) | Golden de bytes, solo para `EscPosRawComandaPrinter` | Cuando se toca el respaldo |
| **Integración — datos** | Migraciones desde cero, UNIQUE de código y teléfono, transacción del contador, snapshots | SQLite en archivo temporal | Cada cambio de datos |
| **Integración — impresión** | Que el camino driver llame a `StartDoc`/`PrintPage` y **nunca** abra diálogo; que el error se propague | `PrintController` falso + `FileComandaPrinter` | Cada cambio de impresión |
| **ViewModel** | Flujo teclado-primero: código → Enter → ítem y total; Tab; confirmar con error de impresora | xUnit sobre ViewModels, sin UI | Cada cambio de pantalla |
| **Manual en el local** | `Imprimir prueba`, ancho y letra reales, reimpresión, apagar impresora | Checklist | Antes de entregar |

**Lo que NO se automatiza en la beta**
- Automatización de UI (UIA): frágil, lenta en esta máquina y de bajo retorno para 8 pantallas.
- Comparación pixel a pixel del PDF: el driver y la térmica rasterizan distinto; se valida visualmente.

**Validación sin impresora real** (no se puede probar con la Ocom antes de la entrega):
1. Golden tests del layout.
2. `FileComandaPrinter` para el recorrido completo pedido → comanda.
3. Impresora local de prueba con puerto `FILE:` para ejercitar el spooler sin diálogo.
4. Checklist de instalación en el local, con `Imprimir prueba` como primera acción.

## Seguridad

- **Autenticación / autorización**: no aplica. Un usuario, una PC, sin red.
- **Validación de entrada**: en `Core` (reglas) y en la UI (formato de código, precio, teléfono).
- **Secretos**: ninguno. No hay claves, tokens ni configuración sensible.
- **Datos**: SQLite local en `%LOCALAPPDATA%\Vargas Pizza\`. La carpeta de backup queda fuera de la carpeta de instalación.
- **Superficie**: cero puertos abiertos, cero llamadas de red. Esto es verificable y debe testearse (test que falla si se agrega un cliente HTTP).

## Variables de entorno

No se usan variables de entorno. Toda la configuración vive en la tabla `configuracion` y se edita desde la app.

| Config | Default | Sensible |
|---|---|---|
| `impresion.impresora_nombre` | (vacío) | No |
| `impresion.modo` | `driver` | No |
| `impresion.ancho_papel_mm` | `80` | No |
| `impresion.tamano_letra_pt` | `9` | No |
| `operacion.hora_corte` | `06:00` | No |
| `backup.carpeta` | `Documentos\Vargas Pizza\Copia de seguridad` | No |
| `backup.retencion_dias` | `30` | No |

## CI/CD

- **`ci.yml`** (push y PR): `dotnet restore`, `dotnet build -c Release`, `dotnet test`.
- **`release.yml`** (tag `v*`): `dotnet publish -c Release -r win-x64 --self-contained`, compilar Inno Setup, subir el `.exe` a GitHub Releases.
- Sin firma de código: el usuario verá la advertencia de SmartScreen. Se documenta el paso a paso en el README de entrega.
