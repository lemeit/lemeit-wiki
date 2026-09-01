# ![Eureka AI](../assets/logos/eureka.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Eureka AI — tutor.lemeit.ar

Tutor digital de ciencias asistido por IA para alumnos de secundario y curso de ingreso universitario. Cubre **Física, Química, Fisicoquímica, Química del Carbono, Biofísica y Matemática** con estilo socrático: concepto inicial breve → guía paso a paso → visualizaciones automáticas (gráficos, diagramas, estructuras químicas) → pregunta de cierre.

- **Sitio principal:** [tutor.lemeit.ar](https://tutor.lemeit.ar) — portal con login Google (Firebase)
- **Landing institucional:** [lemeit.ar](https://lemeit.ar) — página de entrada con CTA al tutor
- **Repositorio:** `lemeit/eureka` (privado — Cloudflare Pages puede leer repos privados gratis; GitHub Pages solo en plan Pro)
- **Versión actual:** Beta 2.0 — junio 2026

## Origen del nombre

Hoy se presenta en el sitio como **Eureka AI**, pero el nombre completo es un acrónimo: **EUREKA** — Entorno de Unificación Racional Estimulando Knowledge Activo. Inspirado en la exclamación griega (εὕρηκα) atribuida a Arquímedes: el clímax del descubrimiento.

| Letra | Concepto |
|---|---|
| E | Entorno digital donde el estudiante explora conceptos |
| U | Unificación de teoría abstracta con fenómenos reales |
| R | Racional — explicaciones lógicamente consistentes |
| E | Estimulando — devoluciones que invitan a seguir explorando |
| K | Knowledge aplicado, no memorización pasiva |
| A | Activo — el estudiante construye su propia comprensión |

## Arquitectura

```
lemeit.ar (landing)             Cloudflare Pages ── repo lemeit/eureka/landing/
        │  CTA "Entrar al Tutor EUREKA"
        ▼
tutor.lemeit.ar (portal, repo privado)
  Browser
   ├── Firebase Auth (Google)
   ├── Firestore: users, queries, messages, sessions, feedback,
   │              cached_responses, compound_images, config/admins
   └── Cloudflare Worker (eureka-proxy → api.lemeit.ar)
        ├── POST /  routing inteligente:
        │      ├── compound_image → PubChem (primario) / RSC ChemSpider (fallback)
        │      ├── multimodal (archivos) → Gemini 2.5 Flash
        │      ├── tutor (≥1024 tok) → Gemini 2.5 Flash (default)
        │      ├── classify (≤100 tok) → Groq llama-3.3 (rápido)
        │      └── fallback a Groq si Gemini falla
        └── GET  /session-info  (IP enmascarada + país)
```

### Decisiones clave

- **Cloudflare Pages** en vez de GitHub Pages: soporta repos privados gratis.
- **Dos modelos de IA**: Groq (texto puro, streaming, gratis) y Gemini (imágenes/PDFs + tutor principal, free tier de 1500 req/día).
- **Origin allowlist en el Worker**: solo `tutor.lemeit.ar`, `lemeit.ar` y `lemeit.github.io` pueden invocarlo, para proteger el cupo de Groq/Gemini.
- **Rate limit por IP** (10 req/min) + cuota por usuario (5 consultas/día).
- **Memoria conversacional** de los últimos 3 turnos, que se resetea al cambiar de materia, cerrar sesión o cambiar el modo de respuesta admin.

## Stack técnico

| Capa | Tecnología |
|---|---|
| Frontend | HTML5 + CSS3 + JS vanilla, single-file, sin build step |
| Markdown | marked.js |
| Fórmulas | KaTeX (`$...$` inline, `$$...$$` bloque) |
| Diagramas conceptuales | Mermaid v10 |
| Gráficos | Chart.js v4 |
| Estructuras químicas | PubChem PUG-REST (primario) + RSC ChemSpider (fallback) |
| Auth | Firebase Authentication v10 (Google OAuth) |
| Base de datos | Cloud Firestore |
| Hosting portal + landing | Cloudflare Pages (git-connected, deploy automático) |
| Proxy API | Cloudflare Workers (CORS, rate limit, routing IA) |
| IA texto | Groq `llama-3.3-70b-versatile`, streaming SSE |
| IA multimodal | Google `gemini-2.5-flash`, imágenes + PDFs hasta 15 MB |

## Materias disponibles

| ID | Materia | Especialización |
|---|---|---|
| `fisica` | Física | Mecánica, electromagnetismo, óptica, termodinámica |
| `quimica` | Química | General + inorgánica, con tag RSC para estructuras |
| `fisq` | Fisicoquímica | Termodinámica química, cinética, electroquímica |
| `carbono` | Química del Carbono | Orgánica, con reglas propias para estructuras y mecanismos |
| `biofisica` | Biofísica | Física aplicada a sistemas biológicos |
| `ingreso` | Ingreso universitario | CBC UBA, UTN, UNLP, UNICEN |
| `matematica` | Matemática | Álgebra, funciones, geometría, cálculo de secundario |

## Diagramas de estructura química

Cuando corresponde, el modelo emite un tag `[COMPOUND_DIAGRAM:nombre]` que el frontend intercepta para mostrar la imagen 2D real de la molécula, con una cascada de dos fuentes:

1. **PubChem (PUG-REST)** — primaria, gratis, sin API key, con cobertura de polímeros (celulosa, almidón, glucógeno) que ChemSpider no tiene.
2. **RSC ChemSpider** — fallback si PubChem falla (requiere `RSC_API_KEY`).

Las imágenes se cachean en Firestore (`compound_images/`) para no repetir la consulta. Cobertura actual: ~290 compuestos + polímeros (aromáticos, glúcidos, lípidos, fármacos, aminoácidos, nucleótidos, química inorgánica, explosivos).

## Sistema de cache del curriculum — estrategia por niveles

El tutor detecta automáticamente cuando una consulta coincide con un ejercicio de la guía del profesor (`detectCurriculumMatch`) y escala la respuesta según cuánto insiste el alumno:

| Nivel | Disparador | Qué recibe el alumno | Tokens del modelo |
|---|---|---|---|
| 0 | Primera consulta del ejercicio | Planteo: concepto + fórmula + primer cálculo + gancho a clases particulares, sin resultado final | 0 si el planteo ya está cacheado; 1 sola generación si no |
| 1+ | El alumno pide ayuda ("no sé", "dame la respuesta"...) | Solución completa verificada, servida directo del cache | 0 siempre |

Este diseño surgió de un problema concreto: dándole la solución completa en el prompt, el modelo la recalculaba mal sistemáticamente (por ejemplo, inventaba 54.327 kJ cuando la respuesta verificada era 5694 kgm) incluso con instrucciones explícitas de copiarla textual — server la solución directo del cache, sin pasar por el modelo, lo resolvió de raíz.

## Método pedagógico — "guía que avanza"

Reemplaza al socrático clásico (una catarata de preguntas que frustraba y quemaba créditos):

- Máximo 2 turnos por ejercicio: la primera respuesta da concepto + primer tramo resuelto + una pregunta puntual accionable; la segunda cierra todo.
- Clasificación en 3 tipos: **A** dato/definición (responde completo), **B** procedimiento (casi completo, deja el último paso), **C** ejercicio con enunciado (guía que avanza).
- Detección de prisa o frustración ("no sé", "apurate") → resuelve completo de una.
- Toggle admin **Socrático / Directo** para testear o comparar ambos modos.

## Administración

Los admins se configuran agregando su email al array `emails` del documento `config/admins` en Firestore — sin tocar código. Las cuentas admin no gastan cupo diario ni ven el modal de fin de cupo.

El versionado usa `tools/bump_version.py` (`show` / `patch` / `minor` / `major`), que actualiza los 3 marcadores de versión en `index.html` de una sola vez.

## Hoja de ruta (próximas mejoras sugeridas)

- Botón de WhatsApp en el gancho a clases, con el ejercicio pre-cargado.
- Pre-generación batch de planteos, con control de calidad desde un panel admin.
- Analytics de ejercicios: más consultados, dónde abandonan los alumnos, tasa de conversión al gancho de clases.
- Sistema de referidos (más consultas por invitar compañeros).
- Modo examen: simulacros cronometrados sin ayuda, corrección al final.
- RAG con embeddings sobre libros de texto (Serway, guías de ingreso).
- Más guías de cátedra: UNSL, CBC UBA, UNLP, UTN.

## Versión y créditos

- **Versión actual**: Beta 2.0 — junio 2026
- **Autoría**: Luciano Lamaita
- **Desarrollo**: Luciano Lamaita con asistencia de Claude (Anthropic)
- **Licencia**: uso educativo, todos los derechos reservados
