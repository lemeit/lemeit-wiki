# Lemeit portal, Eureka presentation and identity

Updated 28 October 2026.

| Destination | URL | Source in lemeit/eureka | Cloudflare Pages project |
|---|---|---|---|
| Lemeit project portal | https://lemeit.ar/ | landing/index.html | lemeit-landing, serves landing/ |
| Eureka AI introduction | https://tutor.lemeit.ar/presentacion/ | presentacion/index.html and eureka-motion.js | eureka, serves repository root |
| Direct tutor access | https://tutor.lemeit.ar/ | index.html | eureka |

Lemeit links to Eureka, Aire Escolar and teaching materials, with secondary links to the Physics concept map, weather and water quality. Its original hand-drawn SVG sensor, light bulb and notebook use cream #FAF7F2, oxide orange #9B3D00 and teal #009688. The personal calligraphic L belongs to Luciano Lamaita, not the Lemeit umbrella brand.

## Eureka: final composition

The introduction retains EUREKA with AI in superscript and “ancora imparo — aún aprendo”. Four fixed areas surround the central explanation and tutor button: tesseract and mass–spring at left, Lorenz and pendulum at right. Each model moves within its own area; panels do not drift or rotate. Small screens show the introduction first, then a two-column model grid.

A pause control stops the motion. Reduced-motion preferences are respected, and evolution stops while the tab is hidden. Small equations, orange markers, trails and state readings accompany each model, without a calculation table or scene selectors.

These are illustrative mathematical models, not measured data: projected 4D hypercube rotations; Lorenz with σ=10, ρ=28, β=8/3 and RK4 integration; ideal frictionless mass–spring x=A cos(ωt); and the ideal nonlinear pendulum θ̈+(g/L)sinθ=0, integrated with RK4 without the small-angle approximation. Unverified scientist quotations were not restored.

## Favicon and retained alternative

**Encuentro** is selected as the Lemeit favicon, at landing/favicon.svg. Thin orange segments meet a teal wave on cream. Arrowheads use automatically oriented SVG markers, enlarged relative to the line. The HTML declares a versioned icon URL to refresh caches.

**Interacción** remains in docs/identidad-lemeit/favicon-interaccion.svg in the private Eureka repository for future use. Its four arrows were corrected too. These are original brand symbols inspired by Feynman diagram notation; no specific physical process is assigned to them. Other project favicons remain unchanged.

## Publishing and verification

Both Lemeit and Eureka use Git-connected Pages projects. Merging main triggers their deployments. Teaching materials are maintained independently in lemeit/aboutme. Environmental apps stay at https://app.lemeit.ar/aq/, /emas/ and /wq/; no environmental data, APIs or app logic changed.

JavaScript syntax, SVG parsing and links were checked, along with desktop scenes and icon previews at 16, 32 and 64 pixels. Exhaustive mobile visual review remains pending. Local 127.0.0.1 previews require a running local server; published Cloudflare sites do not.

Earlier floating and relocating panel experiments are not the final published layout.

## Scroll-based project tour

The portal reads “Ciencias · Educación · Ambiente” and has no top navigation menu. The connected illustration acts as a visual index: sensor → Aire Escolar, light bulb → Eureka, notebook → teaching materials. Selecting a symbol scrolls to its chapter; the external site link appears in that chapter's short description.

Each chapter occupies at least one screen, revealing its enlarged drawing and description as the visitor scrolls. Scrolling remains native; only the scrollbar is visually hidden. IntersectionObserver drives the reveal in landing/lemeit-scroll.js, with readable content when JavaScript is unavailable or reduced motion is requested. Stable anchors are #aire-escolar, #eureka and #materiales. New chapters can extend this pattern.
