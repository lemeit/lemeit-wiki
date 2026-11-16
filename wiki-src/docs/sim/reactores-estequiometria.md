# Reactores: estequiometría, conversión y equilibrio

**Capítulo 7 · Fundamentos de ΣIM.** [Capítulo anterior](bombas-valvulas-presion.md) · [Ejemplos 07 y 08](https://sim.lemeit.ar/ejemplos.html).

## 1. Balance por especie con reacción

En régimen estacionario, el balance de una especie no es simplemente «entrada = salida». Una forma molar útil es **ṅᵢ,s = ṅᵢ,e + νᵢ ξ̇**, para una sola reacción, donde νᵢ es el coeficiente estequiométrico con signo (**negativo para reactivos, positivo para productos**) y ξ̇ es el avance de reacción molar por tiempo.

La suma de caudales molares totales puede cambiar por reacción; la **masa total** y los **elementos químicos** deben conservarse si la reacción y el modelo están correctamente planteados. Comparar especies entrada/salida sin el término de reacción genera falsos avisos de error.

## 2. Reactor de conversión

Para un reactivo A, la conversión se define como **X_A = (ṅ_A,e − ṅ_A,s)/ṅ_A,e**, cuando la base y la interpretación de A son apropiadas. Una conversión de 50 % significa que se consume la mitad del A alimentado, no que el producto necesariamente tenga 50 % másico de A ni que el equilibrio imponga ese valor.

Ejemplo ideal independiente: **A → B**, con 2 mol/s de A puro y X_A = 0,50. Salen **1 mol/s de A** y **1 mol/s de B**. Este ejemplo abstracto sirve para calcular avance, pero no representa por sí solo una química real con propiedades físicas válidas.

## 3. Reactor de equilibrio

Un reactor de equilibrio no prescribe directamente una conversión arbitraria: obtiene un estado compatible con el **modelo de reacción**, la composición inicial, el estado termodinámico y las restricciones de equilibrio. Esto requiere datos químicos y termodinámicos válidos; convergencia no prueba la exactitud de las predicciones.

En ΣIM, la adaptación actual utiliza **una reacción por reactor**, exige coeficientes estequiométricos con reactivos negativos y productos positivos, un compuesto base reactivo, fase de reacción y temperatura de salida. El reactor de conversión requiere, además, porcentaje de conversión.

## 4. Energía y calor de reacción

El calor de un reactor no puede inferirse únicamente del cambio de temperatura sensible. La transformación de especies cambia las contribuciones energéticas; hay que usar la convención y las entalpías de la formulación seleccionada. La potencia del reactor en ΣIM se obtiene de **DeltaQ del motor DWSIM**, que incorpora el efecto de la reacción.

## 5. Actividades de ΣIM

**Ejemplo 07, conversión:** examiná la reacción que produce agua con conversión del **50 %**. Registrá reactivo base, signo de los coeficientes y composición de productos. La energía calculada históricamente fue aproximadamente **−6836,902074 kW**; no la interpretes como resultado de una simple diferencia de temperaturas.

**Ejemplo 08, equilibrio:** estudiá la síntesis de amoníaco. La referencia previa muestra una fracción de amoníaco cercana a **0,311687291752 kg/kg** y Q de **−999,736216 kW** para el caso documentado. No generalices esos valores a otras presiones, composiciones o modelos.

Para ambos casos, identificá qué especies se generan y cuáles se consumen, verificá el balance elemental y justificá por qué no corresponde exigir conservación individual de todas las especies.

## 6. Lo que ΣIM todavía no implementa

No hay CSTR/PFR cinéticos generales ni redes arbitrarias de múltiples reacciones en el editor. No debe presentarse una salida de equilibrio como cálculo de velocidad de reacción, dimensionamiento o tiempo de residencia.

**Referencia:** [catálogo y alcance de reactores](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md). Esta guía es original y utiliza cifras históricas verificadas dentro del alcance documentado.
