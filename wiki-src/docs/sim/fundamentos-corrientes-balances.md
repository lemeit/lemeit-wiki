# Corrientes, estado termodinámico y balances

Esta guía es una explicación original en español para aprender los fundamentos de la simulación de procesos y utilizarlos en **ΣIM**, que se comunica con el motor **DWSIM**. No es una traducción oficial de la documentación de DWSIM ni representa todas las capacidades del programa de escritorio.

**Para comenzar:** [abrir ΣIM](https://sim.lemeit.ar/acceso.html) · [ejemplos del portal](https://sim.lemeit.ar/ejemplos.html) · [catálogo de equipos](https://sim.lemeit.ar/catalogo.html).

## 1. ¿Qué representa una corriente?

Una corriente de materia describe el transporte de uno o más componentes químicos entre equipos. Para calcular sus propiedades debemos distinguir:

- **Composición:** qué sustancias están presentes y en qué proporción; la base másica y la molar no son intercambiables.
- **Caudal:** cuánto material circula por unidad de tiempo, por ejemplo en kg/s o mol/s.
- **Estado termodinámico:** presión, temperatura y otras propiedades que permiten determinar fases y entalpía.
- **Modelo termodinámico:** conjunto de ecuaciones y datos que relaciona esas propiedades para la mezcla estudiada.

Una **corriente de energía** representa una transferencia energética cuando el esquema y el equipo admiten ese vínculo. No transporta por sí misma la masa de una corriente material.

> **Idea central:** especificar la temperatura de una corriente no equivale a especificar su entalpía. La relación depende del fluido, de su estado, de las fases y del modelo elegido.

## 2. Datos de entrada y resultados calculados

En una simulación no se deben imponer arbitrariamente todas las propiedades: algunas se eligen como especificaciones y otras las obtiene el motor. Si un conjunto de datos no define un estado, el problema está subespecificado; si impone condiciones incompatibles, está sobreespecificado.

En ΣIM, para una alimentación habitual se indican temperatura, presión, caudal y composición, además del paquete de propiedades. Las salidas conectadas a los equipos pueden ser resultados del cálculo. Otros modos de especificación deben verificarse en el catálogo y en la interfaz de cada equipo, no suponerse disponibles por pertenecer al DWSIM de escritorio.

## 3. Balance de materia

Para un volumen de control, la forma general es:

**Acumulación = Entradas − Salidas + Generación − Consumo**

En **régimen estacionario**, sin acumulación y sin reacción química:

**Σ caudales másicos de entrada = Σ caudales másicos de salida**

En un equipo sin reacción también puede comprobarse el balance individual de cada componente. En los **reactores**, un componente puede consumirse o generarse; por eso no corresponde declarar un error simplemente porque su caudal de salida difiera del de entrada. La conservación de la masa total y de los elementos químicos sigue siendo relevante.

### Ejemplo didáctico: mezcla de agua

Dos alimentaciones de agua pura ingresan a un mezclador:

| Dato | Corriente 1 | Corriente 2 |
|---|---:|---:|
| Caudal másico | 1 kg/s | 1 kg/s |
| Temperatura | 20 °C | 80 °C |
| Presión absoluta | 101,325 kPa | 101,325 kPa |

Si no hay pérdidas de materia ni reacciones:

**Caudal de salida = 1 kg/s + 1 kg/s = 2 kg/s**

Este resultado se comprueba en el [Ejemplo 01 de ΣIM](https://sim.lemeit.ar/ejemplos.html). No requiere conocer previamente la temperatura final.

## 4. Balance de energía

Para un mezclador estacionario, sin trabajo de eje, con cambios despreciables de energía cinética y potencial, y sin intercambio de calor externo:

**Σ (caudal másico de entrada × entalpía específica de entrada) = Σ (caudal másico de salida × entalpía específica de salida)**

Aquí ***h*** representa la entalpía específica, expresada en unidades consistentes como kJ/kg. Como 1 kg/s × 1 kJ/kg = 1 kW, el producto caudal–entalpía se interpreta como una tasa de transporte energético.

Para el caso de mezcla de agua del Ejemplo 01, el resultado de referencia previamente calculado con DWSIM 10.2.15 es **aproximadamente 50,021387 °C** y **2 kg/s**. La temperatura obtenida no debe reemplazarse por la media aritmética como regla general: la igualdad fundamental del caso es la de *entalpías transportadas*.

En calentadores y enfriadores, el signo de la energía transferida debe leerse según la convención usada. En los ejemplos de ΣIM, el calor positivo es **hacia el fluido**; por eso un enfriamiento tiene potencia térmica negativa.

## 5. Cómo realizar la práctica en ΣIM

1. Ingresar al portal con una sesión válida y abrir **Ejemplo 01 · Mezcla de agua**.
2. Identificar ambas corrientes de entrada, el mezclador y la salida.
3. Revisar unidades, presiones y composición de las alimentaciones.
4. Seleccionar explícitamente el motor disponible (**Motor principal o Motor alternativo**); ΣIM no cambia automáticamente de motor ante fallas.
5. Calcular y comparar el caudal de salida con la suma de entradas.
6. Comparar la temperatura calculada y los valores de entalpía que presenta el resultado; distinguir datos especificados de propiedades calculadas.
7. Cambiar una temperatura de alimentación, volver a calcular y explicar qué se conserva.

**Preguntas para el aula:** ¿por qué la temperatura final no siempre es el promedio de las entradas? ¿Qué cambiaría si los caudales fueran diferentes? ¿Qué información adicional hace falta al mezclar agua y etanol? ¿Por qué un balance individual de componentes requiere tratar de modo especial a un reactor?

## 6. Alcance y fuentes

- Los números de los ejemplos corresponden a **resultados de modelos**, no a mediciones experimentales. La coincidencia entre dos gateways que usan DWSIM no constituye validación experimental independiente.
- ΣIM es un editor web pedagógico y actualmente no implementa todos los equipos, especificaciones ni métodos disponibles en DWSIM.
- **Referencia técnica principal:** [DWSIM Tutorials — Streams, State and Balances](https://dwsim.org/tutorials/en/fundamentals/01-streams-state-and-balances.html), consultada como fuente de conceptos. Esta página ofrece una redacción, organización y actividad didáctica propias.
- **Referencia de práctica:** [ΣIM — ejemplos y resultados](https://sim.lemeit.ar/ejemplos.html).
- **Seguimiento de capacidades:** [roadmap de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md).

### Próximos temas de la serie

**Grados de libertad y especificaciones** → **Modelos termodinámicos** → **Unidades, composición y fases** → **Operaciones unitarias y convergencia**. Cada tema debe vincular fundamentos con una práctica verificable en el portal.
