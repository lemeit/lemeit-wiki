# Diagramas de proceso, corrientes y conexiones

**Capítulo 8 · Fundamentos de ΣIM.** [Capítulo anterior](reactores-estequiometria.md) · [Guía de trabajo](https://github.com/lemeit/lemeit-sim/blob/main/docs/workflow.md).

## 1. Del proceso físico al diagrama

Un diagrama de proceso describe cómo se relacionan corrientes y equipos: las **operaciones unitarias** transforman el estado, la composición, la presión o los caudales; las **corrientes materiales** transportan materia; las **corrientes energéticas** indican transferencia de energía cuando el equipo admite ese vínculo.

Un diagrama no es correcto solo por verse ordenado: la dirección y el tipo de cada conexión deben representar un proceso físico y responder al contrato de puertos del editor.

## 2. Reglas del editor ΣIM

La conexión une una **salida** con una **entrada** compatible, alternando entre un equipo y una corriente; no se conectan directamente dos equipos ni dos corrientes. Un puerto no puede ocuparse más de una vez. El gateway valida nuevamente estas condiciones antes de invocar DWSIM.

| Equipo | Entradas materiales | Salidas materiales | Observación |
|---|---:|---:|---|
| Mezclador | 1–3 | 1 | La salida es resultado de las alimentaciones |
| Divisor | 1 | 2 | Una fracción define el reparto |
| Separador flash | 1 | 2 | Productos vapor y líquido |
| Calentador / enfriador | 1 | 1 | Temperatura objetivo |
| Bomba / válvula | 1 | 1 | Presión de salida prescrita |
| Intercambiador | 2 | 2 | Lados caliente y frío separados |
| Reactor | 1 | 2 | Productos vapor y líquido |

El editor ofrece corrientes de energía y puertos energéticos en los equipos pertinentes; esto no significa que todos los equipos requieran una conexión de energía independiente. La bomba con presión prescrita **no requiere** corriente energética para definir la especificación en este modo.

## 3. Alimentaciones y salidas calculadas

Una corriente material **sin entrada desde otro equipo** es una alimentación y requiere especificar propiedades y composición. La corriente material situada aguas abajo de un equipo debe recuperar sus resultados del cálculo, no volver a imponerse como alimentación independiente. Si se desconecta, su condición de alimentación puede cambiar.

La revisión del proyecto identifica conexiones incompletas y parámetros básicos. El servidor verifica con mayor detalle los tipos, las direcciones, los índices, los componentes y las especificaciones. Los resultados siguen dependiendo del motor DWSIM.

## 4. Orden práctico de armado

1. Definí los componentes y el modelo termodinámico.
2. Agregá las alimentaciones y completá sus datos.
3. Incorporá los equipos y sus corrientes de salida.
4. Conectá puerto por puerto, confirmando el sentido y la naturaleza material/energética.
5. Completá **solo las especificaciones que corresponden al modo activo** de cada operación.
6. Usá Herramientas → Diagnóstico del proyecto; después calculá y compará resultados con balances.

## 5. Recirculaciones: concepto y limitación

Una recirculación devuelve parte del material desde una etapa posterior a una anterior. En simuladores que la admiten, es común elegir una *corriente de corte* (*tear stream*), proporcionar valores iniciales e iterar hasta cumplir una tolerancia de convergencia.

**ΣIM incorpora un primer modo de recirculación material simple**, con validación física y cruzada aún pendientes para los ejemplos 17–19. Los lazos complejos, el cálculo nativo de Recycle y las recirculaciones con reacción requieren una validación diferente. No asumir convergencia por el simple hecho de que la conexión sea aceptada.

## 6. Práctica

Dibujá y construí una alimentación de agua → divisor de dos productos. Completá la fracción de la primera salida y verificá que ambas existan. Eliminá una salida y observá el diagnóstico previo. Restaurala, calculá y comprobá **F = P₁ + P₂**.

Como segunda actividad, armá dos alimentaciones → mezclador → una salida. Antes de calcular, explicá por qué no se prescribe la composición de la salida como si fuera una alimentación nueva.

**Referencias:** [manual de trabajo de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/docs/workflow.md) · [DWSIM — Recycles and Convergence](https://dwsim.org/tutorials/en/fundamentals/03-recycles-and-convergence.html). Guía propia y adaptada a la versión documentada.
