# Especificación Técnica — Correcciones de Interfaz Mobile
## Proyecto: Play 360 (play360.vercel.app)

---

## ⚠️ RESTRICCIÓN CRÍTICA — LEER ANTES DE IMPLEMENTAR

**Todos los cambios especificados en este documento deben aplicarse EXCLUSIVAMENTE a la versión mobile/responsive del sitio.**

- **NO modificar** ningún estilo, layout, comportamiento, posicionamiento ni lógica de la versión de escritorio (desktop), bajo ninguna circunstancia.
- Toda implementación debe estar encapsulada mediante:
  - Media queries (`@media (max-width: 768px)` o el breakpoint que ya use el proyecto para mobile), y/o
  - Detección de dispositivo táctil / viewport (`window.matchMedia`, `ontouchstart`, etc.)
- Antes de tocar cualquier archivo, identificar si los estilos/lógica actuales son compartidos entre desktop y mobile. Si lo son, **refactorizar para separar el comportamiento mobile sin alterar el desktop**, no sobrescribir directamente estilos globales.
- Verificar visualmente en viewport desktop (≥1024px) que no hubo ningún cambio antes de dar por cerrada la tarea.

---

## 1. NAVBAR FLOTANTE (componente de navegación entre slides)

### 1.1 Problema
El navbar flotante (logo "Play 360" + flechas de navegación ← → + contador "X/14") está demasiado compacto/angosto en viewport mobile. Los elementos internos (logo, flechas, contador) se ven apretados y no tienen suficiente espacio horizontal interno (padding/gap).

### 1.2 Fix requerido
- Aumentar el `width` (o `min-width`) del contenedor del navbar flotante **únicamente en mobile**.
- Aumentar el `padding` horizontal interno del contenedor para dar más espacio de "respiro" entre el logo, las flechas y el contador.
- Verificar que el `gap` entre los elementos internos (logo / separador / flecha-izquierda / contador / flecha-derecha) sea consistente y no quede comprimido en pantallas de ~360px–414px de ancho (los anchos típicos de dispositivos Android/iPhone).
- Mantener el navbar centrado horizontalmente y con la posición vertical actual (el usuario indicó que la posición vertical del navbar le gusta, **no moverlo en el eje Y**).

### 1.3 Ícono/elemento erróneo en esquina superior izquierda
- En el **Slide 1** (slide de portada, logo "américa paraguay" + "Play 360"), se reporta la aparición de un ícono/elemento visual no identificado en la esquina superior izquierda de la pantalla, descrito como una "cara triste" o ícono de error/roto.
- **Acción requerida:**
  1. Inspeccionar el DOM/CSS de esa slide en viewport mobile para identificar el origen del elemento (puede ser un emoji, un ícono SVG roto por `alt` faltante, un favicon mal renderizado, o un elemento decorativo mal posicionado que en desktop queda oculto/fuera de viewport).
  2. Una vez identificado, eliminarlo o corregir su renderizado para que no aparezca en mobile.
  3. Si no se logra reproducir el bug al inspeccionar, documentar el hallazgo y solicitar una nueva captura de pantalla con zoom en esa esquina para mayor precisión.

---

## 2. SCROLL VERTICAL DENTRO DE CADA SLIDE (requerimiento ya validado en iteración anterior — incluido aquí para que no se omita)

### 2.1 Problema
En mobile, cuando el contenido de una slide excede el alto de la pantalla, no es posible hacer scroll vertical dentro de esa slide para ver el contenido restante (cards, listas de beneficios, íconos de redes sociales, etc. quedan cortados y no se pueden visualizar).

### 2.2 Fix requerido
- Habilitar `overflow-y: auto` (o `scroll`) en el contenedor de contenido de cada slide, **solo en mobile**, cuando el contenido supere el alto del viewport.
- Agregar `-webkit-overflow-scrolling: touch;` para scroll fluido en iOS.
- Asegurar que el navbar flotante no bloquee la zona de scroll: aplicar `pointer-events: none` al contenedor del navbar y `pointer-events: auto` únicamente a sus elementos interactivos (logo, flechas), para que el gesto de swipe/scroll pase a través de él si el usuario toca esa zona pero quiere scrollear contenido debajo.

---

## 3. DESACTIVACIÓN DE NAVEGACIÓN ENTRE SLIDES POR SWIPE VERTICAL (requerimiento ya validado en iteración anterior — incluido aquí para que no se omita)

### 3.1 Problema
Actualmente, el gesto de swipe vertical (deslizar el dedo hacia arriba/abajo) está vinculado al cambio de slide. Esto entra en conflicto directo con el punto 2 (scroll de contenido) y degrada la experiencia en mobile.

### 3.2 Fix requerido
- Detectar si el dispositivo es mobile (`window.matchMedia("(max-width: 768px)").matches` o `'ontouchstart' in window`).
- Si es mobile: **no registrar** los event listeners de `wheel` / `touchmove` / `touchstart` que actualmente disparan el cambio de slide por gesto vertical.
- Si es desktop: mantener el comportamiento actual sin modificaciones (el swipe vertical debe seguir cambiando de slide en PC).
- La navegación entre slides en mobile debe quedar disponible **únicamente** a través de los botones de flecha (← →) del navbar flotante. Estos listeners de click/tap deben seguir funcionando igual en ambas plataformas, sin tocarlos.

---

## 4. CENTRADO VERTICAL — SLIDE 1 (Portada)

### 4.1 Problema
El bloque de contenido (logo "américa paraguay" + logo "Play 360") está posicionado demasiado abajo en el viewport mobile, generando un espacio vacío excesivo en la parte superior de la pantalla y dejando el contenido descentrado.

### 4.2 Fix requerido
- Ajustar el centrado vertical del contenedor de esta slide en mobile (revisar `justify-content`, `align-items`, `margin-top`/`padding-top`, o el uso de `flex`/`grid` con `place-content: center`).
- El objetivo es que el bloque de logos quede centrado verticalmente en el viewport visible (considerando el espacio que ocupa el navbar flotante), subiendo el contenido respecto a su posición actual.
- No se debe modificar el tamaño de los logos, solo su posición vertical dentro del contenedor.

---

## 5. CENTRADO VERTICAL — SLIDE 14 (última slide)

### 5.1 Problema
Mismo problema que el Slide 1: el contenido está descentrado verticalmente, ubicado demasiado abajo.

### 5.2 Fix requerido
- Aplicar el mismo criterio de corrección que en el punto 4: subir el contenido para lograr centrado vertical real dentro del viewport mobile.

---

## 6. ESPACIO DE SCROLL INSUFICIENTE AL FINAL DE SLIDE (padding-bottom) — POR SLIDE

### 6.1 Problema general
En varias slides, el contenido final (última card, íconos de redes sociales, lista de beneficios, etc.) queda cortado en la parte inferior de la pantalla y no hay suficiente espacio para scrollear hasta verlo completo. Esto está directamente relacionado con el punto 2 (habilitar `overflow-y`), pero requiere además un ajuste de `padding-bottom` específico en el contenedor de contenido de cada slide afectada, para que al llegar al final del scroll el último elemento no quede pegado al borde inferior de la pantalla ni oculto detrás del navbar flotante.

### 6.2 Regla técnica a aplicar
- Para cada slide listada abajo: agregar `padding-bottom` (valor sugerido inicial: `80px`–`120px`, ajustable según altura real del navbar flotante + margen de seguridad) al contenedor interno de contenido scrolleable, **solo en mobile**.
- Esto debe combinarse con el `overflow-y: auto` del punto 2: el padding-bottom debe estar **dentro** del área scrolleable, no fuera de ella, para que efectivamente se pueda hacer scroll hasta revelar ese espacio y ver el último elemento completo.
- **Importante:** este ajuste de espacio inferior no debe generar compresión ni reposicionamiento de los elementos superiores de la slide (títulos, headers). Los títulos deben mantenerse en su posición de diseño original; el problema a resolver es exclusivamente la falta de espacio de scroll al final, no un reordenamiento del contenido existente.

### 6.3 Slides afectadas (requieren `padding-bottom` adicional + scroll habilitado)

| Slide | Contenido | Problema específico |
|---|---|---|
| **Slide 2** | "Nuestro Alcance Digital" | No se pueden ver los íconos de redes sociales (Instagram, Facebook, TikTok, YouTube, X) al final de la slide. |
| **Slide 3** | "Presentando: Play 360" | Falta espacio para scrollear hasta el final del bloque de texto. |
| **Slide 4** | "La agenda de Play 360" | Falta espacio arriba (ver 6.4) y abajo; la card "Cobertura de eventos" queda cortada. |
| **Slide 5** | "Las voces que construyen la industria" | Falta espacio al final para ver completo el video/card embebido. |
| **Slide 6** | "Conducción: Lorena Rojas" | Falta espacio al final para ver completa la bio y los íconos de Instagram/LinkedIn. |
| **Slide 9** | "El valor de estar en" (4 cards: Combate al Juego Ilegal / Juego Responsable / Credibilidad Sectorial / Proyección Regional) | Falta espacio al final para ver completas las 4 cards, especialmente la card "Credibilidad Sectorial" que queda con texto cortado. |
| **Slide 11** | "Plan Presentador" | Falta espacio al final para ver completa la lista "Incluye:" (ítem de entrevista y "Logo en 4 reels del programa"). |
| **Slide 12** | "Plan Auspiciante" | Mismo problema: falta espacio al final para ver completa la lista "Incluye:". |
| **Slide 13** | "Plan Apoyo" | Falta espacio al final de la lista de beneficios. |

### 6.4 Caso especial — Slide 4 (espacio arriba Y abajo)
A diferencia de las demás slides de la tabla anterior, la **Slide 4** requiere además espacio adicional en la **parte superior** del contenedor de contenido (`padding-top`), de manera que:
- El título "La agenda de Play 360" no quede amontonado, recortado o demasiado pegado al borde superior del viewport al iniciar el scroll.
- El usuario pueda scrollear de forma natural sin que el encabezado se sienta "apretado" contra el navbar superior o el borde de la pantalla.
- Esto debe lograrse únicamente con espaciado (`padding`/`margin`), sin alterar el tamaño de fuente, la jerarquía visual ni el orden de los elementos.

### 6.5 Slides que NO requieren cambios (confirmado por el cliente — no tocar)
- **Slide 7** — "Lo que representa" (4 cards: Profesionalización del Sector / Redefiniendo el Sector / Combate al Juego Ilegal / Proyección Regional)
- **Slide 8** — "Oportunidad Única"
- **Slide 10** — Slide de transición/contenido confirmado como correcto

---

## 7. CHECKLIST DE VALIDACIÓN POST-IMPLEMENTACIÓN

Antes de marcar esta tarea como completa, validar en un dispositivo mobile real o emulador (viewport ~360px–414px de ancho):

- [ ] El navbar flotante se ve más ancho y sus elementos internos (logo, flechas, contador) tienen espacio suficiente, sin overlap ni compresión.
- [ ] El ícono/elemento extraño en la esquina superior izquierda del Slide 1 ya no aparece (o se documentó por qué no se pudo reproducir).
- [ ] En cada slide con contenido largo, se puede hacer scroll vertical completo dentro de la slide sin que esto dispare un cambio de slide.
- [ ] El swipe vertical (arriba/abajo) en mobile **no cambia de slide**; solo lo hacen las flechas del navbar.
- [ ] En desktop, el swipe vertical **sigue cambiando de slide** exactamente igual que antes (sin regresiones).
- [ ] Slide 1: contenido centrado verticalmente, subido respecto a la posición original.
- [ ] Slide 14: contenido centrado verticalmente, subido respecto a la posición original.
- [ ] Slides 2, 3, 4, 5, 6, 9, 11, 12, 13: se puede scrollear hasta el final y ver el último elemento completo (íconos, cards, listas) sin que quede cortado por el borde de pantalla.
- [ ] Slide 4 específicamente: el título superior no queda recortado/amontonado al iniciar el scroll.
- [ ] Slides 7, 8 y 10: **sin cambios** respecto a su estado actual.
- [ ] Versión desktop (≥1024px): **sin ningún cambio visual ni funcional** respecto al estado previo a esta tarea.

---

## 8. NOTAS GENERALES PARA LA IMPLEMENTACIÓN

- Usar el breakpoint mobile que ya esté definido en el proyecto (Tailwind, CSS Modules, styled-components, etc.). Si no existe uno consistente, estandarizar a `max-width: 768px` para estos fixes y documentarlo.
- Si el proyecto usa un framework de animaciones/transiciones entre slides (ej. Framer Motion, GSAP, Swiper.js, librería custom), revisar su configuración de `direction`/`axis` para desactivar el eje vertical solo en mobile, en lugar de remover listeners manualmente, si la librería lo permite vía props/config.
- Probar en al menos dos tamaños de viewport mobile distintos (ej. 360x800 y 414x896) para confirmar que los paddings agregados no generan espacios excesivos en pantallas más grandes dentro del rango mobile.
