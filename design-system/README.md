# Design System — Vargas Pizza

**Mode:** off
**Approval:** skipped
**Revision:** n/a (no se creó un sistema de diseño)
**Fecha:** 2026-11-25
**Fuente de la decisión:** elección explícita del usuario en el gate de política visual de la fase de fundación.

## Decisión

El proyecto **no adopta un sistema de diseño formal** ni mockups por change. La app es una herramienta interna de escritorio WPF para un solo operador, sin marca ni requisito de marketing; el valor está en la velocidad de carga y en la legibilidad de la comanda.

Con `Mode: off`, el gate de mockups de `design-mockups` no se dispara en cada change visual. Esta decisión se puede revisar si el cliente pide una identidad visual.

## Convención mínima obligatoria (no es un sistema de diseño)

A pedido explícito del usuario, **desde la primera pantalla** rige esta convención. No es opcional y no se reemplaza por valores sueltos:

1. **Un único `ResourceDictionary` compartido**: `src/VargasPizza.Desktop/Resources/Styles.xaml`. Se importa en `App.xaml` y lo consumen todas las vistas.
2. **Colores base con contraste suficiente**: texto y fondo deben cumplir WCAG AA (≥ 4.5:1 para texto normal). Los colores se declaran como recursos con nombre, nunca como literales en XAML de vistas.
3. **Tipografía grande para mostrador**: tamaño base pensado para lectura a distancia de un operador parado frente a la PC; el tamaño de los ítems de la comanda se configura aparte (impresión).
4. **Foco de teclado bien visible**: todo control enfocable debe tener un estilo de foco de alto contraste, visible sin ambigüedad, porque la carga es teclado primero.
5. **Sin valores sueltos**: colores, tamaños de fuente, márgenes, paddings y radios salen de `Styles.xaml`. Una vista no define colores ni tamaños propios; si falta un valor, se agrega al diccionario compartido.

## Cómo revisar esta decisión

- Cambiar la política: pedido explícito del usuario; se actualiza este README y `visual_design.mode` en `.active-orchestrator-state.json`.
- Cambiar un valor visual: se edita `Styles.xaml`; no se toca cada vista.
- Si el proyecto adopta marca en el futuro, conviene migrar a un `design-system/` completo con tokens y previews.
