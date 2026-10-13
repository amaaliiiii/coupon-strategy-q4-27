# Coupon Visibility E2E Implementation Plan

> Implementar después de aprobar este plan. No crear un segundo prototipo ni mover el journey a otro archivo.

**Goal:** Actualizar los mocks existentes de `#journey` para representar el baseline de visibilidad Coupon MM/HVU desde Home hasta Order Confirmed, empezando por Mass Market.

**Architecture:** Mantener `index.html` como single-file. Centralizar datos y estado en el script existente, renderizar cada etapa desde `screens`/`stageSpecs`, reutilizar componentes visuales de journey y extender `bindPhoneActions` para filtros, navegación, basket, desbloqueo y override exclusivo de Checkout.

**Tech Stack:** HTML, CSS y JavaScript vanilla inline.

**Design reference:** `docs/superpowers/specs/2026-09-09-coupon-visibility-e2e-design.md`

## Global Constraints

- Modificar sólo `index.html`.
- Limitar cambios funcionales y visuales a `#journey` y sus selectores/datos dedicados.
- Mantener Benchmark, `#hub`, Measurement y las demás secciones intactas.
- No modificar ni agregar assets.
- No agregar dependencias, frameworks, build step ni archivos JavaScript/CSS externos.
- Mantener proporción móvil y habilitar scroll interno en todos los teléfonos del journey.
- Conservar el mismo Coupon, tienda, mínimo, subtotal, stacking, ahorro y total entre etapas.
- Cart autoaplica la mejor combinación y no permite cambiarla.
- Checkout conserva la recomendación por defecto y es el único lugar que permite cambiarla.
- No mezclar New User Choose One ni McDonald’s Unlock & Choose con el baseline.
- Agregar sólo interacciones útiles; no agregar animación decorativa.
- Conservar accesibilidad básica: botones semánticos, `aria-label`, foco visible y estados comunicados en texto.

---

## Task 1: Estructura, datos y estado compartido

**Files:**
- Modify: `index.html`

**Outcome:** El journey tiene ocho etapas coherentes (`Home`, `Coupon Hub`, `Search`, `Store`, `Product`, `Cart`, `Checkout`, `Order Confirmed`) y una sola fuente de verdad para datos y estado.

- [ ] **Step 1: Inventariar y aislar selectores del journey**

Antes de editar, localizar los bloques actuales:

```bash
cd /Users/amaaliiiii/Cupones
rg -n 'section class="journey"|const screens =|const stageSpecs =|function renderPersonaStage|function bindPhoneActions' index.html
```

Conservar IDs y selectores usados por otras secciones. Prefijar nuevos estilos y hooks con `journey-` o `data-journey-*` para evitar colisiones con Benchmark y Promo Stacking.

- [ ] **Step 2: Extender la navegación de etapas**

Agregar `Coupon Hub` después de Home y `Order Confirmed` después de Checkout en los tabs MM/HVU. Usar keys estables `hub` y `confirmed` en `screens` y `stageSpecs`. Ajustar el grid responsive de tabs para ocho etapas sin overflow de página.

- [ ] **Step 3: Centralizar fixtures**

Crear en el script un objeto `journeyData` con valores concretos y consistentes:

- Coupon principal: `30% OFF`, máximo `$15.000`, mínimo `$35.000`, vence hoy.
- Store principal: `Home Burgers`.
- Productos: `Classic Burger` por `$26.900` y `Papas` por `$8.900`; juntos forman un subtotal de `$35.800`.
- Stack recomendado: Coupon `−$10.740` + envío gratis `−$4.900`, ahorro total `$15.640`, service fee `$1.800` y total final `$26.860`.
- Alternativa 1: Store Promo `−$6.000` + envío gratis `−$4.900`, ahorro total `$10.900` y total final `$31.600`.
- Alternativa 2: `15% OFF` `−$5.370` + envío gratis `−$4.900`, ahorro total `$10.270` y total final `$32.230`.
- Hub: `30% OFF` en Restaurantes, `$8.000 OFF` en Starbucks, envío gratis en Markets y producto gratis en Turbo, clasificados por vertical y tipo.

Eliminar montos y nombres incompatibles dentro de los templates de `#journey`; todas las etapas deben leer de `journeyData`.

- [ ] **Step 4: Introducir estado local**

Crear `journeyState` con etapa por persona, filtros de Hub, subtotal, mínimo, estado de desbloqueo, stack recomendado y stack seleccionado en Checkout. Añadir funciones pequeñas para derivar `remainingToUnlock`, ahorro aplicado, ahorro proyectado y total final. No persistir en storage.

- [ ] **Step 5: Habilitar viewport móvil scrolleable**

Cambiar sólo el viewport del journey para que `.app-scroll` use `overflow-y:auto`, `overscroll-behavior:contain` y padding inferior suficiente para CTAs fijos. Mantener `.screen` como clip del shell y usar `aspect-ratio:9 / 19.5` o dimensiones equivalentes en desktop y mobile.

- [ ] **Step 6: Verificar estructura y sintaxis**

```bash
cd /Users/amaaliiiii/Cupones
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
HTMLParser().feed(Path("index.html").read_text())
print("HTML parse OK")
PY
python3 - <<'PY'
import re
from pathlib import Path
html = Path("index.html").read_text()
scripts = re.findall(r"<script>(.*?)</script>", html, flags=re.S)
Path("/tmp/coupon-visibility-e2e.js").write_text("\n".join(scripts))
print(f"Extracted {len(scripts)} inline script block(s)")
PY
node --check /tmp/coupon-visibility-e2e.js
rg -n 'data-stage="(home|hub|search|store|product|cart|checkout|confirmed)"' index.html
```

---

## Task 2: Home y Coupon Hub Mass Market

**Files:**
- Modify: `index.html`

**Outcome:** Home introduce Coupons como wayfinder dentro de una experiencia Rappi y el Hub permite explorarlos por relevancia, vertical y beneficio.

- [ ] **Step 1: Reconstruir Home MM**

Reemplazar el hero agregado por un Home Rappi con ubicación, verticales, contenido editorial, Wayfinder Carousel de tickets y cards de tienda. Añadir offer tag amarillo e icono Coupon reutilizable. Mantener el Coupon principal como primer ticket.

- [ ] **Step 2: Añadir pill flotante sobre search**

Ubicar el Coupon más relevante en un pill persistente inmediatamente sobre search. El pill debe abrir el detalle útil del Coupon o navegar a Store; no debe tapar contenido ni bloquear el scroll interno.

- [ ] **Step 3: Conectar `Ver todos` al Hub interno**

Cambiar la acción de Home para llamar `renderPersonaStage("mm", "hub")` y actualizar el tab activo, en vez de desplazar la página a `#hub`.

- [ ] **Step 4: Implementar Coupon Hub**

Renderizar en `screens.hub.mm`:

- carousel superior “Más relevantes para ti”;
- chips independientes para vertical;
- chips independientes para tipo de beneficio;
- inventario de Coupons;
- regreso a Home;
- CTA desde un Coupon relevante hacia Search o Store.

Implementar filtrado desde `journeyData` y re-render parcial del inventario, conservando foco y estados activos.

- [ ] **Step 5: Verificar Home y Hub**

Con el servidor local activo, confirmar en Chrome que Home hace scroll dentro del teléfono, el carousel se desplaza horizontalmente, `Ver todos` abre Hub y ambos grupos de filtros cambian resultados.

```bash
cd /Users/amaaliiiii/Cupones
rg -n 'Wayfinder|Ver todos|Más relevantes para ti|Explorar por vertical|Tipo de beneficio|offer-tag|coupon-icon' index.html
```

---

## Task 3: Search, Store y Product Mass Market

**Files:**
- Modify: `index.html`

**Outcome:** La señal Coupon se mantiene consistente en resultados, tienda y productos, y el mínimo de canasta se puede desbloquear mediante una interacción real.

- [ ] **Step 1: Normalizar Search**

Mantener search, filtros, ETA, rating y delivery. Renderizar en cada resultado elegible el mismo tag amarillo e icono Coupon usados en Home. Añadir interacción de filtro Coupon y acción para abrir la Store principal.

- [ ] **Step 2: Reconstruir Store con estructura Rappi**

Conservar cover, metadata, categorías y lista de productos. Agregar carousel de Coupons aplicables por encima del menú y un módulo de progreso que derive subtotal, mínimo y monto faltante desde `journeyState`.

- [ ] **Step 3: Implementar desbloqueo**

Los botones de agregar producto deben actualizar subtotal y progreso. Al cruzar `$35.000`, cambiar el módulo a `Coupon desbloqueado`, reflejar el beneficio en productos elegibles y actualizar el ahorro derivado. La transición puede usar una transición breve de estado, desactivada por `prefers-reduced-motion`.

- [ ] **Step 4: Actualizar cards y detalle Product**

Mostrar ticket Coupon, precio original tachado, precio impactado/ahorro y condición pendiente cuando corresponda. El CTA de Product debe actualizar el mismo basket y navegar a Cart o regresar a Store de manera explícita.

- [ ] **Step 5: Verificar continuidad**

```bash
cd /Users/amaaliiiii/Cupones
rg -n 'Coupon desbloqueado|remainingToUnlock|minimumBasket|data-journey-add|data-journey-store|data-journey-product' index.html
```

En Chrome: abrir Search → Store, agregar hasta superar el mínimo, confirmar cambio de progreso, abrir Product y comprobar que el precio/ahorro coincide con Store.

---

## Task 4: Cart, Checkout y Order Confirmed

**Files:**
- Modify: `index.html`

**Outcome:** Cart explica la optimización sin selector; Checkout permite el único override; Confirmed cierra el journey con los valores finales.

- [ ] **Step 1: Implementar resumen automático en Cart**

Renderizar la combinación recomendada como resultado, no como opción. Mostrar subtotal, promociones aplicadas, ahorro proyectado y condición pendiente. Recalcular al agregar/quitar productos. No incluir radios, lista de stacks, `Cambiar`, `Ver otras ofertas` ni handlers de selección.

- [ ] **Step 2: Conservar recomendación en Checkout**

Inicializar `selectedCheckoutStack` con `recommendedStack`. Mostrar stacking por línea, ahorro total, fees y total final con los mismos valores de Cart.

- [ ] **Step 3: Implementar override sólo en Checkout**

Añadir `Cambiar combinación` como acción secundaria. Abrir un sheet con alternativas válidas; al elegir una, actualizar selección, breakdown, ahorro y total. Cerrar el sheet y comunicar la selección aplicada.

- [ ] **Step 4: Añadir Order Confirmed**

La arquitectura actual permite agregar esta etapa sin una sección nueva. Crear `screens.confirmed` y `stageSpecs.confirmed`, y conectar el CTA `Pedir` de Checkout. Mostrar la orden estable `#RP-48271`, estado confirmado, combinación aplicada, ahorro final y total. No agregar tracking, mapa ni estados de entrega.

- [ ] **Step 5: Verificar exclusividad del override**

```bash
cd /Users/amaaliiiii/Cupones
python3 - <<'PY'
from pathlib import Path
html = Path("index.html").read_text()
cart = html[html.index("cart: {"):html.index("checkout: {")]
assert "offer-option" not in cart
assert "Cambiar combinación" not in cart
assert "Cambiar combinación" in html
assert "confirmed:" in html
print("Cart override absent; Checkout override and Confirmed present")
PY
```

En Chrome: modificar basket en Cart, continuar a Checkout, comprobar recomendación por defecto, cambiar stack, verificar nuevos totales y finalizar en Order Confirmed.

---

## Task 5: Paridad HVU

**Files:**
- Modify: `index.html`

**Outcome:** HVU cubre las mismas ocho etapas y reglas económicas, con menor prominencia promocional y sin divergencia de estado.

- [ ] **Step 1: Reutilizar datos y componentes**

Construir `screens.*.hvu` con los mismos `journeyData`, cálculos, tienda y basket. Usar variantes de copy/clase para una presentación más curada; no duplicar montos hardcoded.

- [ ] **Step 2: Aplicar diferencias de jerarquía**

Mantener ranking editorial en Home/Search, una señal Coupon secundaria pero visible, confirmación compacta en Store/Product y progresión menos gamificada en Cart.

- [ ] **Step 3: Mantener reglas compartidas**

Cart HVU muestra auto-aplicación sin override. Checkout HVU conserva recomendación y permite cambio como acción secundaria. Order Confirmed refleja la selección final.

- [ ] **Step 4: Verificar paridad**

```bash
cd /Users/amaaliiiii/Cupones
python3 - <<'PY'
from pathlib import Path
html = Path("index.html").read_text()
for persona in ("mm", "hvu"):
    for stage in ("home", "hub", "search", "store", "product", "cart", "checkout", "confirmed"):
        marker = f'data-persona="{persona}" data-stage="{stage}"'
        assert marker in html, marker
print("MM/HVU stage parity OK")
PY
```

Recorrer las ocho etapas HVU en Chrome y comparar subtotal, mínimo, stack recomendado, ahorro y total con MM.

---

## Task 6: Verificación final en Chrome

**Files:**
- Verify: `index.html`
- Verify unchanged: `assets/benchmarks/screens/*`

**Outcome:** El baseline funciona en desktop y viewport estrecho, no altera otras secciones y no contiene placeholders.

- [ ] **Step 1: Ejecutar checks estáticos**

```bash
cd /Users/amaaliiiii/Cupones
python3 - <<'PY'
from html.parser import HTMLParser
from pathlib import Path
html = Path("index.html").read_text()
HTMLParser().feed(html)
for required in ('id="benchmark"', 'id="journey"', 'id="hub"', 'id="measurement"'):
    assert required in html, required
print("Required sections and HTML parse OK")
PY
python3 - <<'PY'
import re
from pathlib import Path
html = Path("index.html").read_text()
Path("/tmp/coupon-visibility-e2e.js").write_text("\n".join(re.findall(r"<script>(.*?)</script>", html, flags=re.S)))
PY
node --check /tmp/coupon-visibility-e2e.js
test -f assets/ObjectSans-Variable.woff2
test "$(ls assets/benchmarks/screens/*.jpg | wc -l | tr -d ' ')" = "12"
! rg -n -i 'TODO|TBD|FIXME|PLACEHOLDER|lorem ipsum' index.html
```

- [ ] **Step 2: Levantar el prototipo**

```bash
cd /Users/amaaliiiii/Cupones
python3 -m http.server 4173
```

Abrir `http://127.0.0.1:4173/index.html#journey` en Google Chrome.

- [ ] **Step 3: Recorrer E2E Mass Market**

Validar en este orden:

1. Home: scroll interno, Wayfinder, pill sobre search, tags amarillos e icono Coupon.
2. `Ver todos`: abre Hub dentro del journey.
3. Hub: filtros por vertical y tipo cambian resultados.
4. Search: filtro Coupon y apertura de Store.
5. Store: agregar productos cambia progreso y desbloquea Coupon.
6. Product: precio y ahorro coinciden; agregar actualiza basket.
7. Cart: mejor combinación automática, proyección y pendientes; ningún override.
8. Checkout: recomendación por defecto, stacking, cambio de combinación y totales reactivos.
9. Order Confirmed: conserva stack, ahorro y total finales.

- [ ] **Step 4: Recorrer E2E HVU**

Repetir las etapas y confirmar paridad de reglas con menor prominencia promocional. Verificar de nuevo que Cart no cambia combinación y Checkout sí.

- [ ] **Step 5: Probar responsive y scroll**

Probar en Chrome a `1440 × 900` y con emulación móvil `390 × 844`. En ambos:

- no hay overflow horizontal de página;
- cada teléfono mantiene proporción móvil;
- el scroll ocurre dentro del teléfono cuando el cursor está sobre él;
- tabs y CTAs permanecen accesibles;
- sheets no salen del viewport del teléfono.

En DevTools Console ejecutar:

```js
getComputedStyle(document.querySelector('#mmStage .app-scroll')).overflowY
```

El resultado debe ser `auto` o `scroll`.

- [ ] **Step 6: Probar regresión de secciones intactas**

Abrir y ejercer tabs/carousels de Benchmark, Promo Stacking y Measurement. Confirmar que no cambió su layout, contenido ni interacción y que las doce capturas siguen cargando sin errores 404.

## Definition of Done

- MM y HVU tienen ocho etapas operativas dentro de `#journey`.
- Todos los criterios por pantalla de la spec están satisfechos.
- Home/Hub implementan la dirección Wayfinder aprobada.
- Store/Product/Cart/Checkout conservan continuidad económica.
- Sólo Checkout permite cambiar combinación.
- Order Confirmed cierra el flujo sin ampliar el producto más allá de confirmación.
- Benchmark y demás secciones permanecen intactas.
- Los checks estáticos pasan y el recorrido Chrome funciona en desktop y móvil.
- No hay placeholders, dependencias nuevas, assets nuevos ni animación decorativa.
