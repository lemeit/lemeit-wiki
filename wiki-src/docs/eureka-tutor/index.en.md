# ![Eureka AI](../assets/logos/eureka.svg){: width="36" style="vertical-align:middle;margin-right:8px" } Eureka AI — tutor.lemeit.ar

AI-assisted digital science tutor for high school students and university-entrance exam prep. Covers **Physics, Chemistry, Physical Chemistry, Organic Chemistry, Biophysics and Math** with a Socratic style: a brief initial concept → step-by-step guidance → automatic visualizations (charts, diagrams, chemical structures) → a closing question.

- **Main site:** [tutor.lemeit.ar](https://tutor.lemeit.ar) — portal with Google login (Firebase)
- **Institutional landing page:** [lemeit.ar](https://lemeit.ar) — entry page with a CTA to the tutor
- **Repository:** `lemeit/eureka` (private — Cloudflare Pages can read private repos for free; GitHub Pages only on the Pro plan)
- **Current version:** Beta 2.0 — June 2026

## Where the name comes from

Today it's presented on the site as **Eureka AI**, but the full name is an acronym: **EUREKA** — Entorno de Unificación Racional Estimulando Knowledge Activo (Environment for Rational Unification Stimulating Active Knowledge). Inspired by the Greek exclamation (εὕρηκα) attributed to Archimedes: the moment of discovery.

| Letter | Concept |
|---|---|
| E | Digital Environment where the student explores concepts |
| U | Unification of abstract theory with real phenomena |
| R | Rational — logically consistent explanations |
| E | Stimulating feedback that invites further exploration |
| K | Knowledge applied, not passive memorization |
| A | Active — the student builds their own understanding |

## Architecture

```
lemeit.ar (landing)             Cloudflare Pages ── repo lemeit/eureka/landing/
        │  CTA "Enter the EUREKA Tutor"
        ▼
tutor.lemeit.ar (portal, private repo)
  Browser
   ├── Firebase Auth (Google)
   ├── Firestore: users, queries, messages, sessions, feedback,
   │              cached_responses, compound_images, config/admins
   └── Cloudflare Worker (eureka-proxy → api-eureka.lemeit.ar)
        ├── POST /  smart routing:
        │      ├── compound_image → PubChem (primary) / RSC ChemSpider (fallback)
        │      ├── multimodal (files) → Gemini 2.5 Flash
        │      ├── tutor (≥1024 tok) → Gemini 2.5 Flash (default)
        │      ├── classify (≤100 tok) → Groq llama-3.3 (fast)
        │      └── falls back to Groq if Gemini fails
        └── GET  /session-info  (masked IP + country)
```

### Key decisions

- **Cloudflare Pages** instead of GitHub Pages: supports private repos for free.
- **Two AI models**: Groq (plain text, streaming, free) and Gemini (images/PDFs + main tutor, 1500 req/day free tier).
- **Origin allowlist on the Worker**: only `tutor.lemeit.ar`, `lemeit.ar` and `lemeit.github.io` can call it, to protect the Groq/Gemini quota.
- **Rate limit per IP** (10 req/min) + per-user quota (5 queries/day).
- **Conversational memory** of the last 3 turns, reset when switching subjects, logging out, or changing the admin response mode.

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | HTML5 + CSS3 + vanilla JS, single-file, no build step |
| Markdown | marked.js |
| Formulas | KaTeX (inline `$...$`, block `$$...$$`) |
| Concept diagrams | Mermaid v10 |
| Charts | Chart.js v4 |
| Chemical structures | PubChem PUG-REST (primary) + RSC ChemSpider (fallback) |
| Auth | Firebase Authentication v10 (Google OAuth) |
| Database | Cloud Firestore |
| Portal + landing hosting | Cloudflare Pages (git-connected, automatic deploy) |
| API proxy | Cloudflare Workers (CORS, rate limit, AI routing) |
| Text AI | Groq `llama-3.3-70b-versatile`, SSE streaming |
| Multimodal AI | Google `gemini-2.5-flash`, images + PDFs up to 15 MB |

## Available subjects

| ID | Subject | Specialization |
|---|---|---|
| `fisica` | Physics | Mechanics, electromagnetism, optics, thermodynamics |
| `quimica` | Chemistry | General + inorganic, with an RSC tag for structures |
| `fisq` | Physical Chemistry | Chemical thermodynamics, kinetics, electrochemistry |
| `carbono` | Organic Chemistry | Organic, with its own rules for structures and mechanisms |
| `biofisica` | Biophysics | Physics applied to biological systems |
| `ingreso` | University entrance prep | CBC UBA, UTN, UNLP, UNICEN |
| `matematica` | Math | High-school algebra, functions, geometry, calculus |

## Chemical structure diagrams

When relevant, the model emits a `[COMPOUND_DIAGRAM:name]` tag that the frontend intercepts to show the molecule's real 2D image, with a two-source fallback chain:

1. **PubChem (PUG-REST)** — primary, free, no API key, with coverage of polymers (cellulose, starch, glycogen) that ChemSpider lacks.
2. **RSC ChemSpider** — fallback if PubChem fails (requires `RSC_API_KEY`).

Images are cached in Firestore (`compound_images/`) so the query isn't repeated. Current coverage: ~290 compounds + polymers (aromatics, carbohydrates, lipids, drugs, amino acids, nucleotides, inorganic chemistry, explosives).

## Curriculum cache system — tiered strategy

The tutor automatically detects when a query matches an exercise from the teacher's guide (`detectCurriculumMatch`) and escalates its response based on how much the student insists:

| Level | Trigger | What the student gets | Model tokens |
|---|---|---|---|
| 0 | First query on the exercise | Setup: concept + formula + first calculation + a hook toward private tutoring, no final answer | 0 if the setup is already cached; 1 generation only if not |
| 1+ | The student asks for help ("I don't know," "give me the answer"...) | Full verified solution, served straight from cache | Always 0 |

This design came out of a concrete problem: when given the full solution in the prompt, the model would systematically recalculate it wrong (for example, inventing 54,327 kJ when the verified answer was 5,694 kgm) even with explicit instructions to copy it verbatim — serving the solution straight from cache, bypassing the model, fixed it at the root.

## Teaching method — "guide that moves forward"

Replaces the classic Socratic method (a cascade of questions that frustrated students and burned through credits):

- Max 2 turns per exercise: the first response gives the concept + the first solved stretch + one actionable, specific question; the second wraps it up.
- Classified into 3 types: **A** fact/definition (answered in full), **B** procedure (almost complete, leaves the last step), **C** exercise with a full prompt (guide that moves forward).
- Detects urgency or frustration ("I don't know," "hurry up") → solves it fully in one go.
- Admin **Socratic / Direct** toggle to test or compare both modes.

## Administration

Admins are configured by adding their email to the `emails` array in the `config/admins` document in Firestore — no code changes needed. Admin accounts don't use up the daily quota or see the end-of-quota modal.

Versioning uses `tools/bump_version.py` (`show` / `patch` / `minor` / `major`), which updates all 3 version markers in `index.html` at once.

## Roadmap (suggested next improvements)

- WhatsApp button on the class-hook, with the exercise pre-loaded.
- Batch pre-generation of exercise setups, with quality control from an admin panel.
- Exercise analytics: most-queried, where students drop off, conversion rate to the class hook.
- Referral system (more queries for inviting classmates).
- Exam mode: timed practice runs with no help, graded at the end.
- RAG with embeddings over textbooks (Serway, entrance-exam guides).
- More course guides: UNSL, CBC UBA, UNLP, UTN.

## Version and credits

- **Current version**: Beta 2.0 — June 2026
- **Authorship**: Luciano Lamaita
- **Development**: Luciano Lamaita with assistance from Claude (Anthropic)
- **License**: educational use, all rights reserved
