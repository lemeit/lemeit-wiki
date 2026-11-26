# Acceso y clases

Funciones documentadas hasta ΣIM 0.51.7.

## Alumnado

Abrir [el ingreso](https://sim.lemeit.ar/acceso.html), elegir **Alumno** e ingresar el código de 12 caracteres. No se necesita correo ni cuenta personal externa. El código es compartido por la clase; una identidad de alumno dentro de esa clase permite organizar los proyectos individuales en la nube.

El pie gris muestra nombre de clase, tiempo restante y **Salir**. La sesión dura hasta cuatro horas o hasta que venza/se desactive la clase, lo que ocurra primero. Si la clase sigue vigente, volver a ingresar renueva la sesión. Antes de salir, pulsá **Guardar en nube**. Elegí un alias y una clave personal si es tu primer guardado. Volvé a ingresar con la misma clase e identidad para retomar el proyecto.

El aviso de acceso finalizado no cierra ni borra el proyecto, pero el servidor ya no permite calcular con una sesión inválida.

## Administración

1. Elegir **Administrador** en el ingreso y validar la contraseña privada.
2. En **Clases y accesos**, escribir el nombre que verá la clase y elegir vigencia: 2, 4, 8 o 24 horas, o 7 días.
3. Pulsar **Crear código** y compartir solo el enlace y ese código con la clase. La vigencia empieza al crearlo.
4. Consultar el listado para volver a copiar códigos activos, revisar vencimientos o desactivar accesos.

Los nuevos códigos se conservan cifrados y solo el administrador puede consultarlos. Los anteriores a 0.17.0 siguen funcionando, pero el listado indica **Anterior · no recuperable** si no tienen copia cifrada. Si se perdió uno anterior, crear un acceso nuevo; la actualización no altera las clases existentes.

Desactivar una clase invalida sus sesiones y pide confirmación. El administrador no muestra contador, aunque su sesión tiene un vencimiento interno de ocho horas.

## Cambiar de acceso y consultar motores

El ingreso avisa si ya hay una sesión abierta. Para cambiar de acceso, cerrarla con **Salir** o con **Cerrar sesión y cambiar acceso**. Seleccionar Administrador no cambia el rol sin validar la contraseña. El panel privado exige permisos en servidor incluso si se conoce su dirección.

**Herramientas → Estado de motores** abre una consulta de motor principal/alternativo; **Administración de clases** abre el panel privado. El monitor del motor elegido también se abre desde el pie. Estas vistas no ejecutan simulaciones ni permiten encender/reiniciar servicios.

Los diálogos se cierran por botón, Escape o clic fuera. Para cambiar de alumno dentro de la clase, usar la opción de cambio de identidad desde la biblioteca de proyectos en nube. Los proyectos se mantienen separados por identidad. Guardar antes de cerrar sesión.

En **Administración → Operación y recuperación**, podés copiar los comandos de diagnóstico, arranque y recuperación para Windows o Debian, o descargar la copia HTML para consulta local. Ejecutá los comandos en el equipo correspondiente. Ver [manual técnico de recuperación](https://github.com/lemeit/lemeit-sim/blob/main/docs/operacion-local.md).

## Documentación técnica

- [Guía de administración](https://github.com/lemeit/lemeit-sim/blob/main/docs/administracion-clases.md).
- [Roles, sesiones y seguridad](https://github.com/lemeit/lemeit-sim/blob/main/docs/acceso-clases-admin.md).
- [Publicación y migraciones](https://github.com/lemeit/lemeit-sim/blob/main/docs/despliegue-automatico.md).

No publicar contraseñas, claves de origen, secretos de sesión ni códigos reales de clases en la wiki o en GitHub.
