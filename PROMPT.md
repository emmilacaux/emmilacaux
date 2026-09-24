# Emmi Lacaux — digital archive

## Contexto para Claude Code

Este es el sitio personal/archivo fotográfico de la artista Emmi Lacaux. Ya está
construido y funcionando como un único `index.html` autocontenido (HTML/CSS/JS
vanilla, sin build step, routing por hash). Quiero que sigas trabajando sobre
este mismo proyecto **respetando exactamente el sistema de diseño ya definido**
— esto NO es un rediseño, es continuación.

Archivos en esta carpeta:
- `index.html` — el sitio completo tal cual está ahora mismo.
- `NewBerolinaMT.ttf` — fuente real del nombre de la artista (Monotype, script
  caligráfico). Ya está embebida en el HTML como base64 dentro de un
  `@font-face`.
- `Generic-G20-FR-Classic-DEMO.otf` — fuente real del cuerpo del sitio (corte
  DEMO, subida por la artista). También embebida como base64.

**Primer paso recomendado**: refactorizar `index.html` para dejar de embeber
las fuentes en base64 dentro del `<style>` y en su lugar cargarlas desde
`/fonts/NewBerolinaMT.ttf` y `/fonts/Generic-G20-FR-Classic-DEMO.otf` vía
`@font-face` normal. El base64 fue necesario porque el sitio se publicaba como
artifact de un solo archivo sin servidor — en Claude Code con un proyecto real
esa restricción no existe, así que separar los archivos hace el HTML muchísimo
más liviano y editable.

---

## Dirección de arte (no cambiar sin pedirlo explícitamente)

**Concepto**: web personal de artista + archivo fotográfico + internet
independiente de los primeros años, reinterpretado con dirección de arte
contemporánea. NO retro, NO Y2K, NO nostálgico, NO plantilla de portfolio, NO
agencia creativa, NO SaaS.

**Paleta — exacta, no tocar**:
```
--bg:    #FFFFFF   (blanco puro, nunca off-white/crema/beige)
--ink:   #111111
--muted: #5E5A57
--line:  #D7D2CE   (solo para hairlines estructurales, uso excepcional)
--red:   #8B1114   (rojo oscuro tipo tinta de imprenta, NO #D71920 brillante)
```

**Tipografía**:
- Nombre de la artista ("Emmi Lacaux"): `New Berolina MT`, cursiva, rojo. Es
  la ÚNICA aparición grande del nombre (top-left del header, y el `<h1>` de
  About/Contact). Nunca se repite como hero gigante en el home.
- Todo lo demás: `Generic G20-FR Classic`. Capitalización natural, SIN
  `text-transform: uppercase`, SIN letter-spacing excesivo — nada de estética
  "fashion brand" o "SaaS interface".
- Tipografía muy pequeña en general (10–14px). Contraste viene de escala,
  espaciado y tipografía, no de más tipografías.
- Ya se probaron y se descartaron: `-webkit-text-stroke` como efecto
  "raster/digital" (se veía o invisible o como un efecto gráfico obvio), y
  relleno blanco + contorno de color tipo subtítulo de video (rompía
  legibilidad sobre fondo blanco). **No reintentar estos efectos** salvo
  pedido explícito.

**Sistema de links — una sola regla, sin excepciones**:
```
normal:  color: #111111; sin subrayado
hover:   color: #8B1114; subrayado rojo de 1px (text-decoration, no border-bottom)
```
Sin transición (`transition: none`), sin estado "activo" persistente en la
nav — solo hover. Aplica a: nav, entrada de archivo del home, links del
índice, "Back to work"/"Next project", links de contacto, "Go to top".

**Header**: `position: fixed`, fondo blanco sólido, sin sombra/blur/borde,
altura mínima. Izquierda: "Emmi Lacaux". Derecha: "Work / About / Contact".

**Fotografías — regla más importante del sitio**:
- SON el trabajo. El diseño se adapta a ellas, no al revés.
- Todas las fotos de una secuencia usan el MISMO ancho horizontal (`sc-l` en
  el sistema actual). La variación viene de alineación (izq/centro/der),
  pausas de espacio en blanco, presencia/ausencia de texto — nunca de tamaño.
- Proporción natural siempre. NUNCA `object-fit: cover` ni recorte forzado.
  Cuando hay `src` real, la imagen se muestra a su alto natural
  (`width:100%; height:auto`), sin caja de aspect-ratio forzada.
- Sin placeholder decorativo: son cajas planas neutras (`#F1EFEC`) que dicen
  "acá va una imagen", sin gradiente ni grano simulando una foto falsa.
- Completamente estáticas: NUNCA fade/slide/scale/parallax/opacity al entrar
  en viewport. Ya existen en la página desde que carga.
- El sistema de "reveal on scroll" fue eliminado del sitio por completo —
  NO reintroducir animaciones de aparición para nada, ni fotos ni texto
  estructural.

**Metadata / rastros de archivo — escasos e inconsistentes a propósito**:
- Contador `01 / 08` fijo abajo a la derecha, muy chico, sin animación.
- Captions muy ocasionales y variadas: algunas fotos sin nada, otras con
  una nota corta ("Ibiza, 2026"), otras solo un número ("04"). NUNCA todas
  las fotos con el mismo patrón de metadata — eso las vuelve "sistema de
  diseño" en vez de archivo personal.
- Footer: "Go to top" (sin flecha) + "Archive / 2026", muy chico, con una
  sola línea horizontal de separación (es la única línea fuera del Index).
- El Index (`/work`) es la excepción: ahí SÍ se usan líneas finas entre
  filas (número / año / categoría / título) porque es, a propósito, un
  índice/documento. No aplicar ese lenguaje de tabla al resto del sitio.

**Textos**: la homepage muestra `01 — Ibiza — Summer 2026` como único link de
archivo — sin "Open", sin flecha, sin lenguaje de CTA. Los textos del
proyecto (descripción, fragmentos) los escribe la artista; no generar copy
nuevo ni "mejorarlo" salvo que se pida.

---

## Arquitectura técnica actual

- SPA de una sola página con routing por hash (`#/`, `#/work`,
  `#/work/:slug`, `#/about`, `#/contact`), sin librerías, sin build.
- Datos de proyectos en un array `PROJECTS` al principio del `<script>` —
  cada proyecto tiene `slug, title, year, date, location, category, medium,
  description, sequence[]`. `sequence` es una lista ordenada de eventos:
  `{type:'image', ratio, scale, align, caption?, src?, alt?, credit?,
  number?}`, `{type:'text', text, align}`, `{type:'label', text}`,
  `{type:'pause', size}`.
- `photoDiv()` ya soporta imágenes reales: pasale `src` (y `alt`) en un item
  del `sequence` y renderiza un `<img>` de verdad con proporción natural en
  vez del placeholder plano.
- Sistema de escalas de ancho: `sc-xs, sc-s, sc-m, sc-l, sc-xl, sc-full`
  (actualmente todo el proyecto de ejemplo usa `sc-l` para mantener ancho
  uniforme, según la regla de arriba).

## Qué sigue (sugerido, no obligatorio)

1. Separar las fuentes a archivos reales (`/fonts/`) en vez de base64.
2. Cargar fotografías reales de Emmi Lacaux en el proyecto "Ibiza — Summer
   2026" usando el soporte `src` que ya tiene `photoDiv()`.
3. Considerar separar el proyecto en varios archivos (`index.html`,
   `styles.css`, `app.js`, `projects.js`) ahora que no hay restricción de
   archivo único — pero mantené el mismo sistema, no lo rediseñes al mover
   el código.
4. Agregar más proyectos al array `PROJECTS` siguiendo la misma estructura
   (writing, video, music según la categoría).

No agregues efectos, secciones, o funcionalidades nuevas sin que se pidan
explícitamente. La prioridad del sitio siempre fue: fotografía > tipografía >
espacio en blanco > detalles de archivo mínimos. "Hacer menos, pero exacto."
