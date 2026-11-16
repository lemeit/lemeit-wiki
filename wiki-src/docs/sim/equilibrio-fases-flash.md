# Equilibrio de fases y cálculos flash

**Capítulo 4 · Fundamentos de ΣIM.** Un cálculo *flash* determina qué fases pueden coexistir y qué composición tiene cada una en condiciones especificadas. El fenómeno no debe confundirse con el recipiente que separa físicamente las fases. [Capítulo anterior](modelos-termodinamicos.md) · [Ejemplos de ΣIM](https://sim.lemeit.ar/ejemplos.html).

## 1. ¿Qué es un equilibrio líquido–vapor?

En equilibrio, cada componente tiene la misma fugacidad en las fases líquida y vapor. En un esquema simplificado se representa con la relación **yᵢ = Kᵢ xᵢ**, donde *xᵢ* e *yᵢ* son fracciones molares en líquido y vapor, y *Kᵢ* depende del sistema, el modelo y el estado termodinámico. No debe asumirse que *Kᵢ* es constante en cualquier condición.

La composición global satisface **zᵢ = (1 − β)xᵢ + βyᵢ**, con β igual a la **fracción molar de vapor**. También se cumple Σxᵢ = Σyᵢ = Σzᵢ = 1. Si el sistema está completamente líquido, β = 0; si está completamente vapor, β = 1. Una mezcla bifásica requiere 0 < β < 1.

**Atención a las bases:** una fracción molar de vapor no coincide, en general, con el cociente entre caudal másico de vapor y caudal másico global. Para pasar de mol a masa se necesitan las masas molares y las composiciones de cada fase.

## 2. Modalidades flash en una alimentación

En ΣIM se admiten pares TP (temperatura y presión), PH (presión y entalpía), PS (presión y entropía), PV (presión y fracción molar de vapor) y TV (temperatura y fracción molar de vapor), según las restricciones del paquete. **PS con IAPWS-IF97 está bloqueado** en el portal por una discrepancia del motor comprobada previamente.

Estos pares no son órdenes para fijar todas las propiedades al mismo tiempo. Con PH, la temperatura es un resultado. Con PV, la temperatura debe calcularse si existe una solución física admisible. En zonas de una sola fase o cercanas al punto crítico, la elección y estabilidad del cálculo merecen especial atención.

## 3. Balance de un separador flash

Para una alimentación F de mezcla no reactiva y productos vapor V y líquido L, en régimen estacionario: **F = V + L** y, para cada componente i, **F zᵢ = V yᵢ + L xᵢ**, siempre usando la misma base en todos los términos. Para una base molar, las expresiones anteriores son coherentes directamente; para masa deben usarse fracciones másicas y caudales másicos.

El recipiente separa dos fases ya determinadas por la termodinámica de la alimentación y por las condiciones del equipo. Un separador adiabático simple no equivale a una columna de destilación: en general, una única etapa de equilibrio no alcanza altas purezas.

## 4. Práctica con ΣIM: ejemplo 11

Abrí el **Ejemplo 11 — Separador flash**, con agua–etanol y NRTL. La referencia histórica de Debian registra aproximadamente **0,638235345 kg/s de vapor** y **0,361757524 kg/s de líquido**. La pequeña diferencia respecto de 1 kg/s es un residuo numérico que debe interpretarse con la tolerancia documentada, no ocultarse. Consultá el [informe específico](https://github.com/lemeit/lemeit-sim/blob/main/docs/flash-0.10.0.md).

**Actividad guiada:**

1. Identificá los dos componentes y el paquete de propiedades.
2. Distinguí la especificación de la corriente de entrada de las propiedades calculadas de los dos productos.
3. Registrá composición y caudal de vapor y líquido con su **base** correspondiente.
4. Calculá el residuo de masa: entrada menos suma de salidas.
5. Explicá por qué la composición del vapor difiere de la composición del líquido sin violar la conservación de materia.

## 5. Errores frecuentes

- Usar porcentajes másicos como si fueran fracciones molares.
- Interpretar β como proporción de masa sin convertir bases.
- Afirmar que un único flash logra una pureza arbitrariamente alta.
- Suponer que convergencia y conservación de masa prueban que el paquete termodinámico representa el experimento.
- Exigir un número de fases no sustentado por las condiciones.

## 6. Preguntas de cierre

¿Por qué el modelo NRTL es una opción de partida para agua–etanol? ¿Qué propiedad cambia al pasar de TP a PH? ¿Qué diferencias hay entre «calcular equilibrio» y «separar las fases»? ¿Qué informe permite conocer las tolerancias del Ejemplo 11?

**Fuente conceptual:** [DWSIM — Simple Flash Drum](https://dwsim.org/tutorials/en/beginner/04-simple-flash-drum.html). Redacción y ejercicios originales. Los números citados son resultados anteriores de modelos, no mediciones nuevas.
