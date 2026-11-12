# ΣIM · Simulación de procesos

[![sitio: sim.lemeit.ar](https://img.shields.io/badge/sitio-sim.lemeit.ar-009688?style=flat-square&labelColor=555555)](https://sim.lemeit.ar/)
[![código: lemeit-sim](https://img.shields.io/badge/c%C3%B3digo-lemeit--sim-009688?style=flat-square&labelColor=555555)](https://github.com/lemeit/lemeit-sim)

Simulador educativo de procesos con interfaz web en español y motor DWSIM. Estado: **en desarrollo**. Revisión documental: 12 de noviembre de 2026. La identidad propia de la aplicación es ΣIM; la paleta de la wiki y la portada no la reemplaza.

## Accesos

- [Simulador](https://sim.lemeit.ar/).
- [Ejemplos y guía de casos](https://sim.lemeit.ar/ejemplos.html).
- [Catálogo de objetos](https://sim.lemeit.ar/catalogo.html).
- [Ficha docente](https://profe.lemeit.ar/projects/sim/).
- [Código, instrucciones y límites vigentes](https://github.com/lemeit/lemeit-sim).

## Qué permite estudiar

Corrientes de materia y energía; mezcladores, divisores, calentadores y enfriadores; mezclas multicomponente y reactores de conversión o equilibrio dentro del alcance documentado. Los ejemplos incluyen condiciones, consignas y resultados calculados para comparar balances.

El alcance actual es estacionario, sin recirculaciones. La interfaz no reproduce todas las funciones del editor de DWSIM. Los ejemplos son resultados de modelos: no son mediciones experimentales ni una validación independiente del motor.

## Recorrido de trabajo

1. Abrir un ejemplo o preparar equipos y corrientes.
2. Elegir compuestos, modelo de propiedades y condiciones.
3. Seleccionar manualmente un motor disponible, Windows o Debian.
4. Ejecutar y revisar los mensajes y balances antes de interpretar el resultado.
5. Guardar el proyecto y conservar sus condiciones para reproducirlo.

La selección manual no implica conmutación automática. Si el motor no está disponible, abrir el editor y navegar por el catálogo no genera resultados de cálculo.

## Arquitectura y publicación

El navegador muestra el diagrama y los resultados. El cálculo depende de un motor DWSIM externo; no se realiza como la compilación local de documentos de TeX. El repositorio público es `lemeit/lemeit-sim`, separado de `aboutme`, `lemeit/eureka` y `lemeit/simus`.

La publicación del portal utiliza Cloudflare Workers Builds según la configuración del proyecto. La versión visible se comprueba en **Ayuda → Acerca de**; no se deduce solo del número del README o de una fusión en GitHub. Las instrucciones de operación, selección de motores y pruebas están en el repositorio. Esta documentación no configura ni pone en marcha motores privados.

## Nombre, diseño y licencias

Nombre exacto: **ΣIM**, sigma griega mayúscula seguida de I y M latinas. Bajada: **Simulación de procesos**. Reemplaza el nombre inicial SIMPROX, manteniendo dominio y repositorio. La aplicación conserva su identidad técnica propia; la portada Lemeit utiliza un croquis ilustrativo de equipos y corrientes, no un flowsheet validado.

El proyecto usa DWSIM y declara GPL-3.0-or-later; las atribuciones y condiciones se conservan en su repositorio. No es un producto oficial de DWSIM.

Es distinto de [Bestiario · Seres Térmicos](https://profe.lemeit.ar/projects/simus/bestiario/), que combina relato y modelos físicos.
