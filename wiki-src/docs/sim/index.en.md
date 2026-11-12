# ΣIM · Process simulation

ΣIM is an educational process simulator with a Spanish-language web interface and an external DWSIM engine. It is **under development**. Documentation reviewed 12 November 2026.

## Links

- [Simulator](https://sim.lemeit.ar/).
- [Examples](https://sim.lemeit.ar/ejemplos.html) and [object catalogue](https://sim.lemeit.ar/catalogo.html).
- [Teaching-site presentation](https://profe.lemeit.ar/en/projects/sim/).
- [Public source and technical documentation](https://github.com/lemeit/lemeit-sim).

## Scope and workflow

Connect material and energy streams, mixers, splitters, heaters and coolers. Select components, property models and conditions; use conversion or equilibrium reactors within the documented limits. Guided cases support comparison of mass and energy balances.

The current scope is steady state without recycle loops. Cases contain calculated model results, not experimental measurements or independent physical validation. The web editor does not reproduce the full DWSIM desktop interface.

The browser displays the flowsheet and results; an external DWSIM engine performs calculations. Users manually choose an available Windows or Debian engine. There is no automatic failover. Browsing the editor or catalogue does not mean a calculation has run.

## Architecture and publication

Source lives in `lemeit/lemeit-sim`, separately from `aboutme`, `lemeit/eureka` and `lemeit/simus`. Cloudflare Workers Builds publishes the portal according to its project configuration. Check the version actually loaded in **Ayuda → Acerca de**; a README version or merged commit alone does not prove a completed deployment. Motor operation and checks are documented in the repository. This wiki does not configure or start private engines.

## Identity and licensing

The exact name is **ΣIM**: Greek capital sigma followed by Latin I and M. Its subtitle is «Simulación de procesos». It replaces the initial SIMPROX name while preserving domain and repository addresses. The application keeps its own technical visual identity; the Lemeit landing page uses a hand-drawn process illustration, not a validated flowsheet.

The project uses DWSIM and declares GPL-3.0-or-later; see the repository for complete attribution and conditions. It is not an official DWSIM product. It is separate from [Bestiary · Thermal Beings](https://profe.lemeit.ar/projects/simus/bestiario/).
