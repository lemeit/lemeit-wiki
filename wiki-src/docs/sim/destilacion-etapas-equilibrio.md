# Destilación y separación por etapas

**Nivel intermedio · Capítulo 13.** Un separador flash y una columna de destilación no son equivalentes. El primero distribuye fases para un estado de equilibrio; la columna permite múltiples contactos vapor–líquido, con reflujo, condensador, rehervidor y especificaciones de diseño.

[Equilibrio de fases y flash](equilibrio-fases-flash.md) · [Modelos termodinámicos](modelos-termodinamicos.md)

## 1. Una etapa de equilibrio

Para un sistema binario idealizado escribimos:

\[
y_i=K_i x_i
\]

donde \(x_i\) y \(y_i\) son fracciones molares en las fases líquida y vapor, y \(K_i\) depende de la temperatura, presión y modelo de propiedades. En una mezcla real como agua–etanol, el modelo NRTL puede ser más pertinente que la idealidad, dentro de su dominio de parámetros.

La condición de equilibrio **no implica** que ambas fases tengan igual composición. Una separación útil exige una diferencia de volatilidades.

## 2. Balance de una columna

Tomamos el conjunto de columna, condensador y rehervidor como un único volumen de control, con una alimentación \(F\), destilado \(D\) y fondos \(B\). En régimen estacionario:

\[
F=D+B
\]

\[
Fz_i=Dx_{D,i}+Bx_{B,i}
\]

Estas ecuaciones relacionan caudales y composiciones, pero **no** determinan por sí mismas el número de etapas, el reflujo, el consumo térmico ni la ubicación de la alimentación.

## 3. Ejemplo de balance binario resuelto

Se alimentan **100 kmol/h**, con fracción molar del componente más volátil \(z_F=0{,}40\). Se desea obtener destilado \(x_D=0{,}90\) y fondos \(x_B=0{,}10\). Estos valores son **especificaciones de un ejercicio de balances**, no resultados de una columna calculada.

\[
D+B=100
\]

\[
0{,}40(100)=0{,}90D+0{,}10B
\]

Sustituimos \(B=100-D\):

\[
40=0{,}9D+0{,}1(100-D)=0{,}8D+10
\]

Por tanto **D = 37,5 kmol/h** y **B = 62,5 kmol/h**. La comprobación del componente más volátil es \(33{,}75+6{,}25=40\ \mathrm{kmol/h}\).

**Advertencia:** este cierre no demuestra que una columna física alcance esas purezas; todavía faltan el equilibrio de fases, las etapas y las cargas térmicas.

## 4. Variables de operación

- **Relación de reflujo:** cambia la circulación interna y habitualmente el compromiso entre separación y energía.
- **Número de etapas:** influye en la separación alcanzable.
- **Presión:** altera el equilibrio y las temperaturas.
- **Condición y posición de alimentación:** afectan los perfiles internos.
- **Condenser y rehervidor:** establecen intercambios térmicos que deben calcularse y verificarse.

Cuando se especifican demasiadas variables independientes, la columna puede estar sobreespecificada; cuando faltan, el sistema queda indeterminado.

## 5. Limitación del simulador

**ΣIM no dispone actualmente de un editor de columnas de destilación.** El Ejemplo 11 es un **flash de una etapa**, no una destilación por múltiples etapas. Para reproducir una columna usar DWSIM de escritorio y documentar versión, paquete de propiedades, etapas, presión y parámetros de convergencia.

## 6. Actividad de diseño

Con los datos del ejemplo, cambiar \(x_D\) a 0,95 manteniendo \(F=100\), \(z_F=0,40\) y \(x_B=0,10\). Calcular \(D\) y \(B\) solo por balances; luego debatir por qué una pureza mayor podría aumentar las exigencias de separación y energía.

**Resultado analítico:** \(D=(40-10)/(0{,}95-0{,}10)=35{,}2941\ \mathrm{kmol/h}\) y \(B=64{,}7059\ \mathrm{kmol/h}\). No confundir con una simulación rigurosa.

**Referencia conceptual:** [DWSIM — Intermediate Tutorials: Distillation Column](https://dwsim.org/tutorials/en/intermediate/index.html). Guía original con cálculos de balances independientes del motor.
