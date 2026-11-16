# Cinética química y dimensionamiento de reactores

**Nivel intermedio · Capítulo 14.** Los reactores de conversión y equilibrio disponibles en ΣIM sirven para explorar la estequiometría y el estado de salida, pero **no son modelos generales de cinética ni calculan por sí mismos el tamaño de un CSTR o PFR**.

[Reactores y estequiometría](reactores-estequiometria.md)

## 1. Tres preguntas diferentes

- **Estequiometría:** ¿qué especies se consumen y producen?
- **Equilibrio:** ¿cuál es la composición permitida por la termodinámica?
- **Cinética:** ¿a qué velocidad avanza la reacción y cuánto volumen o tiempo se requiere?

Una conversión fijada como entrada no explica qué volumen de reactor logra esa conversión. La composición de equilibrio tampoco proporciona necesariamente la velocidad.

## 2. Ejemplo de reacción de primer orden

Consideremos \(A\to B\), fase líquida con densidad y caudal volumétrico constantes. Sea \(-r_A=kC_A\), con \(k=0{,}20\ \mathrm{min^{-1}}\), y supongamos operación isotérmica.

Para un reactor tubular ideal de flujo pistón (PFR):

\[
V=\frac{\dot V}{k}\ln\left(\frac{1}{1-X}\right)
\]

Para un reactor de mezcla perfecta continuo (CSTR):

\[
V=\frac{\dot V X}{k(1-X)}
\]

Los modelos provienen del balance estacionario junto con la ley de velocidad y requieren sus hipótesis de mezcla y transporte.

## 3. Ejercicio resuelto: comparar volúmenes

Si el caudal volumétrico es \(\dot V=2\ \mathrm{m^3/min}\) y la conversión objetivo \(X=0{,}80\):

**PFR:**

\[
V=\frac{2}{0{,}20}\ln(5)=16{,}0944\ \mathrm{m^3}
\]

**CSTR:**

\[
V=\frac{2(0{,}80)}{0{,}20(0{,}20)}=40\ \mathrm{m^3}
\]

Este resultado es **analítico y didáctico**. No ha sido obtenido ni validado mediante ΣIM. El PFR requiere menor volumen que el CSTR en este caso particular de cinética positiva de primer orden; no es una regla universal para todas las reacciones.

## 4. Efectos térmicos y selectividad

Cuando la velocidad depende de la temperatura, la ley de Arrhenius suele representarse como \(k(T)=A\exp[-E_a/(RT)]\). En reacciones exotérmicas, un aumento de temperatura puede acelerar la reacción y modificar el equilibrio; en reacciones múltiples la selectividad puede depender de la trayectoria térmica y de mezcla. Dimensionar sin balance energético puede producir errores grandes.

## 5. Qué permite practicar ahora

En los ejemplos **07 y 08** de ΣIM se pueden comparar un reactor de conversión y uno de equilibrio según su contrato actual, con una reacción por equipo y salidas vapor/líquido. Registrar la composición calculada, el balance elemental y el resultado energético.

**No presentar** un valor de conversión especificado en ΣIM como predicción de cinética ni inferir volumen o tiempo de residencia a partir de él.

## 6. Actividades

1. Repetir el cálculo PFR y CSTR con \(X=0,50\). **Resultados:** PFR ≈ 6,9315 m³; CSTR = 10 m³.
2. Explicar por qué \(k\) debe tener unidades coherentes con el tiempo.
3. Identificar dos hipótesis que invalidan el ejemplo si el sistema real cambia mucho de volumen.
4. Diferenciar \(X\), rendimiento y selectividad en reacciones múltiples.

**Referencia:** [DWSIM — Reaction Systems](https://dwsim.org/tutorials/en/intermediate/index.html). Ejercicios y derivación originales; los reactores cinéticos quedan fuera del editor ΣIM actual.
