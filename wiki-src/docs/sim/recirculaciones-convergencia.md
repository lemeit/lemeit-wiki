# Recirculaciones y convergencia de procesos

**Nivel intermedio · Capítulo 12.** Hasta aquí estudiamos diagramas sin ciclos. En numerosos procesos reales una fracción del producto retorna a una operación anterior: recuperación de solventes, reciclado de reactivos y recirculación de corrientes de proceso. Este capítulo explica su cálculo desde los balances y la matemática, distinguiendo el balance analítico de un primer soporte de recirculación simple que existe en el gateway. La validación física de los Ejemplos 17–19 todavía está incompleta.

[Itinerario de ΣIM](index.md) · [Diagramas y conexiones](diagramas-conexiones.md)

## 1. Por qué aparece una incógnita adicional

En una cadena abierta calculamos las corrientes siguiendo el orden de equipos. En un circuito cerrado, el caudal de una alimentación depende de una corriente posterior, y ésta a su vez depende de la alimentación. Las ecuaciones deben resolverse simultáneamente o mediante iteraciones.

Una **corriente de corte** (*tear stream*) reemplaza provisionalmente una conexión de retorno por una estimación. El solucionador recorre el resto del diagrama, recalcula la corriente y compara lo calculado con lo supuesto.

## 2. Ejemplo analítico: mezcla y purga

Entran **100 kg/h** de una sustancia conservativa al proceso. A la salida de una operación, una fracción **r = 0,80** del caudal vuelve a la entrada, mientras que el resto se purga. Suponemos régimen estacionario, sin reacción, sin acumulación ni pérdidas.

Definamos **F** como caudal total que ingresa a la operación y **R** como recirculación:

```text
Fresca 100 kg/h ──┐
                  ├── Mezclador ── F ── Operación ──┬── Purga (1-r)F
            R ────┘                                │
                  └──────────────── rF ◀────────────┘
```

El modelo simplificado tiene las relaciones:

\[
F=100+R,\qquad R=0{,}80F
\]

Por sustitución, \(F=100/(1-0{,}80)=500\ \mathrm{kg/h}\), \(R=400\ \mathrm{kg/h}\) y la purga es \(100\ \mathrm{kg/h}\). **La purga iguala la alimentación fresca**, como exige el balance global. Que \(F\) sea cinco veces la alimentación fresca no significa que se cree masa: circula varias veces el mismo material.

## 3. Iteración de punto fijo

Comenzamos suponiendo \(R_0=0\) y repetimos:

\[
F_k=100+R_k,\qquad R_{k+1}=0{,}8F_k
\]

| Iteración | R supuesto (kg/h) | R calculado (kg/h) | Residuo absoluto |
|---|---:|---:|---:|
| 0 | 0 | 80 | 80 |
| 1 | 80 | 144 | 64 |
| 2 | 144 | 195,2 | 51,2 |
| 3 | 195,2 | 236,16 | 40,96 |

El límite es 400 kg/h. Como el factor de contracción es 0,8, la iteración sencilla converge, aunque lentamente. En sistemas no lineales el proceso puede oscilar o divergir: el resultado depende de los métodos numéricos, las estimaciones iniciales y la física representada.

## 4. Residuos y criterios de parada

Un criterio puede basarse en \(|R_{k+1}-R_k|\), acompañado de un residuo relativo y del cumplimiento de balances. Una variación pequeña entre iteraciones **no basta** si el sistema tiene muchas variables o ecuaciones mal condicionadas.

Es útil guardar: corriente de corte, unidades, estimación inicial, tolerancia, iteraciones, balance global, método de convergencia y cualquier advertencia.

## 5. Práctica y alcance en ΣIM

**Actividad analítica:** cambiar la fracción de recirculación a 0,50 y 0,90 y recalcular \(F=100/(1-r)\). Comprobar que las purgas siguen siendo 100 kg/h. Explicar por qué aumenta el caudal interno cuando \(r\) se acerca a 1.

**En ΣIM:** existe un modo inicial de lazo material simple con cálculo iterativo y corriente de corte; los Ejemplos 17–19 y el bloque Recycle nativo no deben presentarse como equivalentes o plenamente validados en Windows y Debian. Comprobar estado del gateway, convergencia y residuos. Para configuraciones avanzadas, utilizar DWSIM de escritorio con modelos y tolerancias explícitos.

## 6. Preguntas

1. ¿Cuál es la diferencia entre caudal fresco y caudal interno?
2. ¿Qué se estima al elegir una corriente de corte?
3. ¿Por qué el balance total debe incluir únicamente las corrientes que atraviesan la frontera del sistema?
4. ¿Qué sucede en el ejemplo si \(r=1\) y se mantiene la alimentación fresca positiva?

**Respuestas:** (1) solo la primera ingresa desde fuera; (2) las variables que definen el retorno; (3) el reciclado interno se cancela; (4) no existe solución estacionaria finita bajo esas hipótesis.

**Referencia conceptual:** [DWSIM — Recycles and Convergence](https://dwsim.org/tutorials/en/fundamentals/03-recycles-and-convergence.html). Redacción, ejemplo y ejercicios originales.
