# Coupon Visibility E2E Redesign

## Objetivo

Rediseñar los mocks E2E de `#journey` para demostrar cómo un Coupon visible y consistente acompaña la compra completa, desde discovery hasta Order Confirmed, con una experiencia Rappi reconocible y con paridad funcional entre Mass Market (MM) y High Value User (HVU).

## Alcance aprobado

El alcance actual es el flujo base de visibilidad MM/HVU, empezando por Mass Market. La implementación inmediata debe actualizar los mocks existentes dentro de `#journey`.

Quedan para una fase posterior, como journeys separados:

- **New User · Choose One:** registro → ticket de bienvenida → elegir tres tiendas → activar tres Coupons → los Coupons aparecen en el Home Wayfinder.
- **McDonald’s · Unlock & Choose:** un Coupon activo desbloquea cinco productos y el usuario elige uno.

La variante “Coupon activo desbloquea cinco productos y user elige uno” no debe mezclarse con el baseline Mass Market. Los modelos conceptuales existentes fuera de `#journey` pueden permanecer como referencia, pero no deben convertirse en etapas ni estados del baseline.

## Estado actual inspeccionado

`index.html` es una implementación single-file con HTML, CSS y JavaScript inline. La sección `#journey` contiene segmentos separados de MM y HVU con seis tabs (`Home`, `Search`, `Store`, `Product`, `Cart`, `Checkout`), templates en `screens`, metadatos en `stageSpecs`, render dinámico con `renderPersonaStage` e interacciones delegadas por `bindPhoneActions`.

Brechas frente al rediseño aprobado:

- Home MM usa un hero agregado, no el Wayfinder Carousel aprobado, y envía a un `#hub` separado.
- Coupon Hub no forma parte de los tabs del journey y prioriza estados de inventario (`Available`, `Used`, `Expired`) en vez de exploración por vertical, tipo de beneficio y relevancia.
- Search muestra señales promocionales, pero no normaliza el tag amarillo y el icono Coupon.
- Store no tiene carousel de Coupons ni una transición interactiva clara entre monto pendiente y desbloqueo.
- Product comunica elegibilidad, pero no muestra ticket Coupon ni precio/ahorro impactado.
- Cart enfatiza el desbloqueo y no resume de forma explícita la mejor combinación automática, el ahorro proyectado y las condiciones pendientes.
- Checkout ya permite cambiar la oferta, pero el copy y la jerarquía deben dejar inequívoco que es el único punto del baseline donde se cambia la combinación.
- No existe una etapa `Order Confirmed` en `screens` o `stageSpecs`.
- `.app-scroll` usa `overflow:hidden`, por lo que los teléfonos del journey no tienen scroll interno aunque sus shells ya tienen dimensiones móviles.

Assets inspeccionados:

- `index.html` referencia `assets/ObjectSans-Variable.woff2`.
- Benchmark referencia doce capturas bajo `assets/benchmarks/screens/` para DiDi, Zomato, Keeta y DoorDash.
- Las capturas verificadas conservan contenido real de benchmark y deben seguir aisladas de los mocks generados en `#journey`.
- El rediseño del journey debe construirse con HTML/CSS/JS inline; no requiere assets nuevos ni debe modificar rutas o contenido de Benchmark.

## Flujo baseline Mass Market

### 1. Home

El Home debe parecer literalmente Rappi y conservar su jerarquía de navegación, verticales, contenido y search. La capa Coupon se integra mediante:

- Wayfinder Carousel de Coupons con tickets concretos y CTA `Ver todos`.
- `Ver todos` abre la etapa Coupon Hub dentro del journey.
- Pill flotante inmediatamente sobre search con el Coupon más relevante.
- Home internamente scrolleable dentro del teléfono.
- Cards de tiendas con offer tag amarillo e icono Coupon.

Interacciones útiles: scroll interno, desplazamiento horizontal del carousel, apertura de Hub y navegación a una tienda elegible. No se agregan loops, parallax, pulsos ni animaciones decorativas.

### 2. Coupon Hub

El Hub es una etapa de exploración, no un selector de combinación para la orden. Debe incluir:

- Carousel superior de Coupons más relevantes.
- Filtro por vertical.
- Filtro por tipo de beneficio.
- Inventario filtrable que conserva valor, alcance, condición y vigencia visibles.
- Acción de regreso a Home y acción útil hacia Search o Store.

### 3. Search

Los resultados elegibles deben mostrar un tag amarillo consistente y el icono Coupon. La señal debe convivir con nombre, ETA, rating y delivery sin sustituirlos. Los controles de búsqueda y filtros deben ser utilizables y el contenido debe admitir scroll interno.

### 4. Store

Store debe sentirse como una tienda Rappi real: cover, metadata, categorías y productos. Debe sumar:

- Carousel de Coupons aplicables.
- Gamification/progress según monto mínimo de canasta.
- Estado inicial con monto pendiente.
- Acción de agregar producto que actualiza la canasta.
- Transición funcional a estado desbloqueado al cumplir el mínimo.
- Beneficio reflejado en los productos elegibles.

La transición de desbloqueo comunica un cambio real de estado; no es una animación decorativa.

### 5. Product / offer cards

Las cards y el detalle de producto elegible deben incluir:

- Ticket o marcador Coupon.
- Precio original cuando aplique.
- Precio impactado o ahorro atribuible.
- Condición pendiente si el beneficio aún no está desbloqueado.
- CTA de agregar que alimenta el estado del Store/Cart.

No se promete como ahorro aplicado lo que todavía depende de una condición.

### 6. Cart

Cart muestra automáticamente:

- La mejor combinación disponible.
- Ahorro aplicado y ahorro proyectado.
- Condiciones pendientes y monto faltante, si existen.
- Recalculo al cambiar la canasta.

Cart no presenta selector, radios ni alternativas para cambiar la combinación. Puede ofrecer acciones para completar la condición, pero nunca autoagrega productos.

### 7. Checkout

Checkout conserva por defecto la recomendación calculada previamente y muestra:

- La combinación recomendada como aplicada.
- El breakdown de promo stacking.
- Ahorro total y total final.
- Una acción secundaria para comparar y cambiar la combinación.

Checkout es el único punto del baseline donde el usuario puede cambiar la combinación. Al elegir otra opción se actualizan selección, ahorro y total.

### 8. Order Confirmed

La inspección confirma que puede añadirse reutilizando el modelo `screens`/`stageSpecs` y los componentes existentes, sin crear otra sección ni ampliar de forma material la complejidad. Por ello, el baseline implementará esta etapa para confirmar:

- Orden creada.
- Ahorro final.
- Combinación aplicada.

## Paridad HVU

HVU debe cubrir las mismas etapas implementadas para MM y reutilizar el mismo lenguaje visual Rappi, componentes y datos base. La diferencia es de intensidad y jerarquía:

- MM hace el inventario, la progresión y el ahorro más prominentes.
- HVU mantiene ranking y contenido editorial, presenta señales Coupon más curadas y conserva auto-aplicación.
- Cart HVU tampoco permite cambiar combinación.
- Checkout HVU es igualmente el único lugar con override.

Paridad no significa duplicar markup divergente: los datos o variantes de persona deben alimentar componentes compartidos siempre que sea posible.

## Modelo de estado e interacciones

El prototipo puede mantener estado local en JavaScript, sin persistencia:

- `activePersona`: `mm` o `hvu`.
- `activeStage`: etapa visible por persona.
- `activeHubVertical` y `activeBenefitType`.
- `basketSubtotal`, `minimumBasket`, `remainingToUnlock` y `couponUnlocked`.
- `recommendedStack` y `selectedCheckoutStack`.

Los datos de Coupons, tiendas, productos y combinaciones deben centralizarse en objetos/arrays del script y ser consumidos por templates, evitando valores incompatibles repetidos en varias pantallas. Home, Store, Product, Cart y Checkout deben mantener el mismo Coupon, tienda, mínimo, subtotal, ahorro y total a través del baseline.

## Constraints globales

- Modificar únicamente el contenido, estilos y comportamiento necesarios para `#journey` en `index.html`.
- Mantener Benchmark y todas las demás secciones visual y funcionalmente intactas, incluido `#hub` existente salvo que sólo se retire su acoplamiento desde el nuevo journey; no reutilizarlo como sustituto del Hub integrado.
- No agregar dependencias, frameworks, build step ni archivos de runtime.
- Mantener HTML/CSS/JS en el single-file existente.
- Usar proporción móvil consistente, viewport recortado por el shell y scroll vertical interno.
- Mantener el scroll de la página independiente del scroll del teléfono.
- Aplicar estilo Rappi literal: fondo claro, coral Rappi, amarillo para offer tags, cards blancas, radios amplios, jerarquía compacta, iconografía Coupon consistente y patrones reconocibles de Home/Search/Store/Cart.
- Conservar foco visible, botones semánticos, labels accesibles y targets utilizables.
- Implementar sólo interacciones que cambien navegación, filtros, canasta, elegibilidad, selección o totales.
- Evitar animación decorativa; respetar `prefers-reduced-motion`.
- No introducir New User Choose One ni McDonald’s Unlock & Choose en el baseline.
- No crear ni modificar assets para los mocks E2E; preservar referencias existentes.

## Criterios de aceptación por pantalla

### Home

- El teléfono tiene proporción móvil y scroll interno operativo.
- Se reconoce como Home de Rappi antes de identificar el módulo Coupon.
- Existe Wayfinder Carousel con al menos dos tickets y CTA `Ver todos`.
- `Ver todos` navega al Coupon Hub del mismo journey.
- Existe pill del Coupon más relevante sobre search.
- Las tiendas elegibles muestran tag amarillo e icono Coupon.

### Coupon Hub

- Es una etapa seleccionable del journey MM y tiene regreso útil.
- El carousel de más relevantes aparece arriba del inventario.
- Vertical y tipo de beneficio son filtros separados y operativos.
- Cambiar un filtro actualiza estados activos y resultados visibles.

### Search

- Los resultados Coupon usan tag amarillo e icono consistente.
- Search y filtros responden a interacción.
- La lista conserva metadata Rappi y scroll interno.

### Store

- Incluye carousel de Coupons y menú de productos.
- El progreso muestra mínimo, subtotal y monto faltante coherentes.
- Agregar productos puede cruzar el mínimo y cambia el estado a desbloqueado.
- Productos elegibles reflejan el beneficio y su impacto.

### Product

- La oferta muestra ticket Coupon, precio original y precio/ahorro impactado.
- Las condiciones pendientes se distinguen del ahorro aplicado.
- Agregar producto actualiza el estado compartido que usa Cart.

### Cart

- Muestra automáticamente la mejor combinación sin selector.
- Distingue ahorro aplicado, ahorro proyectado y condiciones pendientes.
- Recalcula subtotal, elegibilidad y ahorro después de una acción útil.
- No existe control para cambiar combinación.

### Checkout

- La recomendación llega seleccionada por defecto.
- El stacking y sus aportes son legibles.
- `Cambiar combinación` abre alternativas y sólo aquí permite override.
- Cambiar opción actualiza selección, ahorro y total.

### HVU

- Tiene paridad de etapas con MM.
- Usa los mismos datos y reglas económicas del baseline.
- Mantiene señales más curadas sin ocultar el Coupon aplicable.
- Cart no ofrece override y Checkout sí.

### Order Confirmed

- Es alcanzable desde el CTA final de Checkout.
- Confirma ahorro y combinación sin introducir lógica nueva de negocio.

## Fuera de alcance

- Implementación del flujo New User.
- Implementación de elegir uno entre cinco productos desbloqueados.
- Cambios a elegibilidad real, APIs, pricing o persistencia.
- Cambios a Benchmark, Measurement, mecanismos, promo stacking u otras secciones.
- Nuevos assets, librerías o infraestructura.
