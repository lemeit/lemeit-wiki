# Access and classes

Portal 0.17.2 · reviewed 16 November 2026. Menu and button names below match the Spanish interface.

## Students

Open [sign-in](https://sim.lemeit.ar/acceso.html), select **Alumno** and enter the 12-character class code. No email or individual account is needed. A shared class code does not identify individual students.

The grey footer shows the class name, remaining access and **Salir**. A session lasts up to four hours, limited by the class expiry/deactivation. Sign in again to renew a session while the class remains active. Save the project file before signing out or signing in again.

An access-ended notice does not close or erase the project, but the server rejects calculations from an invalid session.

## Administrators

1. Select **Administrador** and validate the private password.
2. In **Clases y accesos**, enter the name students will see and select a validity period: 2, 4, 8 or 24 hours, or 7 days.
3. Choose **Crear código** and share only the simulator link and that code. Validity starts when the code is created.
4. Use the list to copy active codes again, review expiry or deactivate access.

New codes are retained in encrypted form and are only retrievable by administrators. Codes created before 0.17.0 still work, but the list marks them **Anterior · no recuperable** if no encrypted copy exists. Create new class access if such a code was lost; the update does not alter existing classes.

Deactivating a class invalidates its sessions and asks for confirmation. Administrators see no countdown, although their session has an internal eight-hour expiry.

## Switching access and checking engines

The sign-in page indicates an existing session. Close it with **Salir** or **Cerrar sesión y cambiar acceso** when switching access. Selecting Administrator does not change the role without validating the password. The private panel enforces permissions on the server, even when its URL is known.

**Herramientas → Estado de motores** opens the read-only Windows/Debian monitor. **Administración de clases** opens the private panel. The selected engine monitor is also available from the footer. These views do not run simulations or remotely start/restart services.

Dialogs close with a button, Escape or an outside click. Save the project before signing out.

## Technical references

- [Administration guide, Spanish](https://github.com/lemeit/lemeit-sim/blob/main/docs/administracion-clases.md).
- [Roles, sessions and security, Spanish](https://github.com/lemeit/lemeit-sim/blob/main/docs/acceso-clases-admin.md).
- [Deployment and migrations, Spanish](https://github.com/lemeit/lemeit-sim/blob/main/docs/despliegue-automatico.md).

Do not publish real passwords, origin keys, session secrets or class codes in the wiki or GitHub.
