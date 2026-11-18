# Análisis de sensibilidad y escenarios

**Nivel intermedio · Capítulo 15.** La simulación no sirve solamente para obtener un resultado: también permite estudiar **cómo cambia** una respuesta al variar un supuesto, una condición de alimentación o una especificación. Este capítulo desarrolla un procedimiento reproducible sin atribuir a ΣIM una herramienta de optimización que todavía no tiene.

[Modelos termodinámicos](modelos-termodinamicos.md) · [Diagnóstico y validación](diagnostico-validacion.md)

## 1. Variable independiente y respuesta

En un calentador, la temperatura objetivo puede ser un dato independiente; la potencia requerida, una respuesta. En una bomba, la presión prescrita y la eficiencia son datos, mientras que el trabajo del motor es un resultado. En un divisor, la fracción de reparto es la especificación y los caudales de salida son resultados.

**No variar dos variables a la vez** si se desea atribuir el efecto a una sola. Mantener constantes el modelo, la base de composición, las unidades y las demás entradas.

## 2. Diferencias finitas

Una sensibilidad local aproximada puede calcularse con:

\[
\frac{\partial Y}{\partial X}\approx\frac{Y(X+\Delta X)-Y(X-\Delta X)}{2\Delta X}
\]

La fórmula es una aproximación. Para sistemas con cambios de fase o discontinuidades, la sensibilidad puede ser poco estable y la elección de \(\Delta X\) importa.

## 3. Ejercicio resuelto sin motor: divisor

Con una alimentación de 2 kg/s y una fracción de salida \(f\), el caudal del producto 1 es \(m_1=2f\), por lo que:

\[
\frac{dm_1}{df}=2\ \mathrm{kg/s}
\]

| f | m₁ (kg/s) | m₂ (kg/s) |
|---:|---:|---:|
| 0,30 | 0,60 | 1,40 |
| 0,50 | 1,00 | 1,00 |
| 0,70 | 1,40 | 0,60 |

Para comprobar el método de diferencia central en \(f=0,5\) y \(\Delta f=0,1\): \((1,2-0,8)/(0,6-0,4)=2\ \mathrm{kg/s}\). **Los valores son analíticos**, no datos tomados de una corrida de DWSIM.

## 4. Diseñar escenarios de forma honesta

Preparar una tabla con identificador, fecha, motor elegido (Motor principal o Motor alternativo), versión del gateway, paquete termodinámico, compuesto(s), parámetro modificado, entradas, resultados y residuos. Guardar un JSON del proyecto por escenario.

En ΣIM, el usuario puede ejecutar y comparar escenarios de manera **manual**. La existencia de exportaciones de resultados no implica que exista optimización, barrido automático, estimación de incertidumbres ni cálculo formal de derivadas.

## 5. Cambios de modelo e incertidumbre

Cambiar NRTL por otro paquete no es simplemente cambiar un número: cambia las ecuaciones constitutivas. Tratar esa comparación como una **sensibilidad al modelo** distinta de la sensibilidad a variables operativas.

La diferencia entre dos predicciones no constituye un intervalo de confianza. Para cuantificar incertidumbre se requieren datos de parámetros, su variabilidad y un modelo estadístico apropiado.

## 6. Práctica con ΣIM

Con el **Ejemplo 04, divisor**, guardar un escenario base con reparto 70/30 y dos variantes, 60/40 y 80/20. Predecir analíticamente los productos y comprobar los balances después de cada cálculo. Registrar si ambas computadoras producen el mismo resultado dentro de tolerancias declaradas, sin afirmar validación experimental por esa coincidencia.

**Ampliación conceptual:** comparar el deber térmico de un calentador al modificar gradualmente la temperatura objetivo. Distinguir la respuesta calculada de la hipótesis simplificada de calor específico constante.

**Referencia conceptual:** [DWSIM — Tutorials](https://dwsim.org/tutorials/). Desarrollo y ejercicios originales.
