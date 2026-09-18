# Lemeit Projects Documentation

Parent wiki for the **lemeit.ar** projects: technical documentation, usage guides and development log, all in one place. Each project has its own section in the menu; this front page is just the entry point.

## Documented projects

| | Project | What it is | Site | Section |
|---|---|---|---|---|
| ![EMA](assets/logos/ema.svg){: width="28" } ![AQ](assets/logos/aq.svg){: width="28" } ![WQ](assets/logos/wq.svg){: width="28" } | Environmental Monitoring Network | Three sibling environmental monitoring portals (air, weather, water) on Cloudflare, unified under a single domain (`app.lemeit.ar`) since September 2026 | [aq](https://app.lemeit.ar/aq/) · [emas](https://app.lemeit.ar/emas/) · [wq](https://app.lemeit.ar/wq/) | [See section](red-ambiental/index.md) |
| ![Eureka AI](assets/logos/eureka.svg){: width="28" } | Eureka AI | AI-assisted Socratic science tutor, for high school and university entrance-level courses | [tutor.lemeit.ar](https://tutor.lemeit.ar) | [See section](eureka-tutor/index.md) |
| ![Profe](assets/logos/profe.svg){: width="28" } | Profe Lamaita | Personal teaching site: Physics notes (Bigler translation), course materials, and a Quartz/Obsidian concept map | [profe.lemeit.ar](https://profe.lemeit.ar) | [See section](profe-lemeit/index.md) |

New projects get added as new sections as they're documented — this table and the side menu are updated each time.

## About this wiki

- Built with [MkDocs](https://www.mkdocs.org/) + [Material](https://squidfunk.github.io/mkdocs-material/), the same stack used by the [DVBA](https://github.com/lemeit/DVBA) wiki (`wiki-src/` → automatic build via GitHub Actions → `site/` → GitHub Pages).
- Each page's source lives under `wiki-src/docs/` in the [`lemeit/lemeit-wiki`](https://github.com/lemeit/lemeit-wiki) repo, organized in one folder per project.
- All the content is public: there's nothing to hide on the technical or development side. The idea is that anyone — a colleague, a student, someone from the local government, or my own future self a year from now — can understand what each project does, what it's built with, and how to use it, without having to ask for an explanation privately.
- Updated by hand after each relevant work session on any of the projects — there's no automation yet that generates it from the commits on its own.
- **Note**: most of this wiki is also available in Spanish (the default language, and the language most of the content is written and maintained in first). A few pages — like the project development logs — are Spanish-only for now; if you land on one of those from the English menu, you're seeing the original Spanish version.

## How to add a new project to this wiki

1. Create a new folder under `wiki-src/docs/<project-name>/` with at least one `index.md`.
2. Add the section to the `nav:` block in `wiki-src/mkdocs.yml`.
3. Add the corresponding row to the table above.
4. Commit and push to `main` — the build and deploy to `wiki.lemeit.ar` happen automatically.
