# ΣIM · Process simulation

ΣIM is an educational process simulator with a Spanish-language web interface and DWSIM 10.2.15. Verified portal: **0.47.4**, reviewed **24 November 2026**. The application is published and remains under development; it does not reproduce the entire DWSIM desktop editor.

## Links and documentation

- [Sign in](https://sim.lemeit.ar/acceso.html).
- [Class access and administration](acceso.md).
- [User guide](https://sim.lemeit.ar/ayuda.html), [example library](https://sim.lemeit.ar/ejemplos.html) and [catalogue](https://sim.lemeit.ar/catalogo.html). Portal resources require a valid session.
- [Technical index](https://github.com/lemeit/lemeit-sim/blob/main/docs/README.md), [release history](https://github.com/lemeit/lemeit-sim/blob/main/docs/CHANGELOG.md) and [recorded checks](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md).
- [Teaching-site presentation](https://profe.lemeit.ar/en/projects/sim/).

## Class access

Students use a temporary 12-character class code, without email or individual accounts. The footer shows the class, remaining access time and sign-out. Administrators use a separate password and a private panel to create, inspect and deactivate class access; they do not see a countdown.

New active class codes remain available to copy from the private list after reloading. Codes created before 0.17.0 still work, but cannot be recovered if they were not retained. The server validates access and roles; selecting an option on the sign-in form does not grant administrator privileges.

**Estado de motores** is a read-only availability monitor for signed-in users. **Administración de clases** requires an administrator session. The portal cannot remotely start or restart engines. See the [access guide](acceso.md).

## Scope and workflow

The editor supports material/energy streams, mixers, splitters, heaters/coolers, vapour–liquid flash separation, pumps, valves, a two-stream heat exchanger, and conversion/equilibrium reactors within documented limits. Components, composition and thermodynamic models are configured per project.

The example library includes introductory problems and additional recycle cases undergoing validation. Tools cover units, mass/molar bases, CSV/Excel exports and process review. The integrated guide returns to the same project; dialogs close with a button, Escape or an outside click.

The scope is steady state. Initial simple material recycle support is available in the gateway, but convergence and cross-engine validation of examples 17–19 are still incomplete. Kinetic CSTR/PFR models, columns and other engine operations still need adaptation and validation. Example results are calculated models, not experimental measurements or independent physical validation.

1. Sign in with the class code and check the footer identity.
2. Create a project with compounds/models or explicitly open an example.
3. Add equipment and streams, connect ports and enter conditions.
4. Manually select the primary or alternative engine, check availability and calculate.
5. Review calculated streams, balances and the step-by-step process explorer; save individual projects to the class cloud library, or download JSON and export tables/PDF as needed.

The 0.47.4 interface includes categorized menus, dockable panels, on-diagram notes, project metadata, PDF sheets and individual student cloud projects. The energy-report clarification is [under review](https://github.com/lemeit/lemeit-sim/pull/135). There is no automatic failover. A valid session can prepare and save projects while an engine is unavailable, but cannot obtain new calculation results. Save before signing out or signing in again.

## Architecture and verification

The browser edits the flowsheet. A Cloudflare Worker validates session/role and forwards requests to private the primary or alternative engine gateways; DWSIM runs outside the browser. The computers, gateways and tunnels must be available. Publishing the portal does not start them.

Access/interface checks and local in-memory database walkthroughs are not new engine simulations. Calculation reports retain their date and engine; Example 13 has a documented entre motores comparison in 0.14.0. No simultaneous classroom load test or experimental validation is claimed.

Source lives in `lemeit/lemeit-sim`. Workers Builds publishes according to the project configuration. Check the actually loaded version in **Ayuda → Acerca de** and the successful deployment, rather than inferring it from a merged commit. This wiki does not configure secrets or start private engines.

## Identity and licensing

The exact name is **ΣIM**: Greek capital sigma followed by Latin I and M, with the subtitle «Simulación de procesos». The domain and repository retain their original addresses. The application keeps its own black/white/red design; this wiki uses its own identity.

The project uses DWSIM and declares GPL-3.0-or-later; see the repository for attribution and conditions. It is not an official DWSIM product and is separate from [Bestiary · Thermal Beings](https://profe.lemeit.ar/projects/simus/bestiario/).
