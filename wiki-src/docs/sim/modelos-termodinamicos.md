# Selección de modelos termodinámicos en simulación de procesos

Una simulación puede cerrar perfectamente sus balances y aun así predecir mal la separación de fases, la densidad o la entalpía si utiliza un modelo de propiedades inadecuado. Este capítulo explica cómo elegir un **paquete de propiedades** según sustancias, presión, temperatura, equilibrio de fases y datos disponibles. Es una guía didáctica propia para [ΣIM](https://sim.lemeit.ar/), basada en las referencias públicas de DWSIM, **no una traducción oficial**.

[Capítulo 1: corrientes y balances](fundamentos-corrientes-balances.md) · [Capítulo 2: grados de libertad](grados-libertad-especificaciones.md) · [Ejemplos de ΣIM](https://sim.lemeit.ar/ejemplos.html)

## 1. ¿Qué resuelve un paquete de propiedades?

El modelo utiliza la identidad de los componentes, la composición y dos especificaciones intensivas independientes para estimar propiedades y, cuando corresponde, calcular el equilibrio entre fases. Algunas salidas relevantes son:

- Fracción de vapor y composiciones de las fases.
- Densidad, entalpía y entropía.
- Relaciones entre presión, temperatura y equilibrio de fases.
- Propiedades energéticas utilizadas por calentadores, bombas, intercambiadores y reactores.

**Importante:** resolver ecuaciones de balances no equivale a validar la predicción termodinámica. Además, que un paquete aparezca en el catálogo de DWSIM no garantiza que posea parámetros adecuados para la mezcla elegida.

## 2. Tres familias para empezar

### Tablas y formulaciones especializadas

**IAPWS-IF97** está diseñado para propiedades de **agua pura y vapor de agua**. No sirve para describir mezclas agua–etanol. En ΣIM, el gateway restringe este paquete a un único componente llamado `Water`. Por una discrepancia previamente verificada, la combinación flash **PS** está inhabilitada con IAPWS-IF97 dentro de ΣIM.

### Modelos de coeficientes de actividad

**NRTL** y **UNIQUAC** se usan habitualmente para describir desviaciones respecto de la idealidad en mezclas líquidas, especialmente sistemas polares como agua y alcoholes. Los parámetros de interacción binaria son relevantes y deben verificarse.

**UNIFAC** y sus variantes estiman interacciones a partir de contribuciones de grupos funcionales. Son alternativas predictivas cuando no hay parámetros binarios adecuados, pero no sustituyen una contrastación con datos experimentales.

### Ecuaciones de estado

**Peng–Robinson (PR)** y **Soave–Redlich–Kwong (SRK)** son ecuaciones de estado ampliamente utilizadas en gases y mezclas de hidrocarburos. Resultan especialmente pertinentes cuando las condiciones de presión y el equilibrio vapor–líquido de fluidos no polares son importantes. Sus parámetros de interacción también afectan los resultados.

No existe un modelo universalmente mejor: importa **qué sistema** se describe y **para qué propiedad**.

## 3. Tabla orientativa de selección

| Sistema que queremos representar | Primera opción de estudio | Precaución principal |
|---|---|---|
| Agua pura, calentamiento, vapor | IAPWS-IF97 | Solo `Water` en ΣIM |
| Agua + etanol | NRTL | Comprobar parámetros de interacción y equilibrio líquido-vapor |
| Otras mezclas líquidas polares | NRTL o UNIQUAC | Comparar con datos y validar el rango de temperatura |
| Mezclas orgánicas sin parámetros binarios confiables | UNIFAC / Modified UNIFAC | Predicción aproximada: comprobar grupos y dominio |
| Gases e hidrocarburos ligeros | Peng–Robinson | Verificar parámetros, fases y presión |
| Hidrocarburos: análisis alternativo | SRK | Comparar con PR y datos de referencia |
| Agua con sales o electrolitos | Modelo especializado para sales/electrolitos | **No asumir** que NRTL común representa especiación iónica |

Esta tabla es una **orientación inicial**, no una validación automática de la selección.

## 4. ¿Cómo se selecciona en ΣIM?

1. Crear un proyecto y elegir **todos los componentes** necesarios, incluidos productos de reacción cuando corresponda.
2. Escoger un paquete **disponible en el catálogo del motor conectado**. El listado efectivo depende del DWSIM instalado.
3. Definir una alimentación: caudal, base de composición, fracciones que sumen uno y un par flash activo (por ejemplo, TP).
4. Revisar si el paquete es pertinente para las sustancias y el intervalo de presión y temperatura.
5. Calcular con el motor Motor principal o Motor alternativo **elegido manualmente**; ΣIM no cambia de motor automáticamente.
6. Contrastar resultados: estado de fases, balances y propiedades plausibles. Si importa la precisión, comparar contra datos experimentales o una referencia independiente.

**Alcance real:** ΣIM no expone en su interfaz todas las opciones de configuración de parámetros binarios ni todas las extensiones comerciales de DWSIM. El gateway valida que el modelo esté disponible y restringe IAPWS-IF97 a agua pura, pero **no verifica por sí solo la calidad de sus predicciones**.

## 5. Caso A — Agua pura: ¿por qué IAPWS-IF97?

Los [Ejemplos 01 y 02](https://sim.lemeit.ar/ejemplos.html) usan agua pura: una mezcla de 1 kg/s a 20 °C con 1 kg/s a 80 °C, y un calentador de 1 kg/s que lleva agua de 20 °C a 60 °C. Son buenos casos de partida porque el agua cuenta con una formulación específica.

Resultados previamente documentados de DWSIM:

- Mezclador: **2 kg/s**, **50,021387 °C** de salida.
- Calentador: aproximadamente **+167,209679 kW** transferidos al fluido.

**Actividad:** explicar por qué el cálculo elemental de temperatura media (50 °C) no tiene por qué coincidir exactamente con una evaluación de entalpías. Indicar qué variables controla el operador y cuáles calcula el motor.

## 6. Caso B — Agua y etanol: por qué importa NRTL

En el [Ejemplo 05 de ΣIM](https://sim.lemeit.ar/ejemplos.html), la composición de agua–etanol exige reconocer que el líquido puede comportarse de manera no ideal. NRTL permite modelar esas desviaciones siempre que los parámetros y el dominio sean adecuados.

- Se conserva la masa total y de cada especie cuando solo se mezclan las corrientes.
- La distribución entre vapor y líquido, la temperatura de equilibrio y la entalpía pueden depender fuertemente del paquete.
- Un valor calculado por NRTL no prueba por sí mismo la exactitud frente a un experimento.

**Actividad:** construir un esquema sencillo con agua y etanol, seleccionar NRTL y registrar temperatura, presión, composición, fase y balance. Repetir con otra composición y analizar si los resultados cambian de modo razonable. No atribuir una temperatura no comprobada al motor.

## 7. Caso C — Gas e hidrocarburos: Peng–Robinson y SRK

Para una mezcla hipotética de metano y etano, los modelos PR y SRK son puntos de partida razonables. El problema deja de ser el de una mezcla acuosa polar y pasa a requerir un tratamiento apropiado de un sistema de hidrocarburos.

**Actividad propuesta (no validada como nuevo ejemplo del portal):** seleccionar los componentes reales del catálogo, fijar una presión y una temperatura compatibles con el sistema, calcular con PR y registrar la fracción de vapor y la densidad. Repetir con SRK **solo si ambos modelos están disponibles**, comparar y discutir el efecto del modelo. Evitar declarar ganador a uno por converger más rápido.

## 8. Un diagnóstico termodinámico útil

| Observación | Hipótesis para revisar | Acción |
|---|---|---|
| Fracción de vapor inesperada | Par flash, presión, temperatura o paquete | Confirmar unidades, composición y dominio |
| Densidad o entalpía anómala | Fases o propiedades mal representadas | Comparar con referencia fiable |
| Una mezcla polar se trata como ideal | Interacciones no modeladas | Evaluar NRTL/UNIQUAC |
| Dos modelos dan resultados distintos | Sensibilidad de la predicción | Contrastar datos experimentales y parámetros |
| El motor calcula pero un resultado parece imposible | Error físico o de especificación | No dar por validado el proceso; revisar supuestos |

Una comprobación de **grados de libertad** responde si el conjunto de ecuaciones está especificado; el **paquete termodinámico** determina cómo se calculan propiedades y fases; la **validación** pregunta si esos resultados representan aceptablemente el sistema real. Son tres preguntas diferentes.

## 9. Ejercicios de repaso

1. ¿Por qué IAPWS-IF97 es apropiado para agua pura, pero no para agua–etanol?
2. ¿Qué limitación tiene elegir NRTL sin comprobar los parámetros de interacción?
3. ¿Cuándo sería razonable probar Peng–Robinson en lugar de un modelo de actividad?
4. Si dos paquetes convergen pero predicen fracciones de vapor diferentes, ¿cómo decidir cuál usar?
5. ¿Por qué un balance de masa correcto no demuestra que las densidades y las entalpías sean correctas?

**Respuestas orientativas:** (1) la formulación especializada corresponde a agua pura; (2) pueden introducirse errores significativos de equilibrio; (3) gases o hidrocarburos con dominio adecuado; (4) confrontando los modelos con datos, parámetros y rango operativo; (5) conservación de masa y propiedades termodinámicas son verificaciones distintas.

## 10. Referencias y límites

- [DWSIM — Property Packages Guide](https://dwsim.org/tutorials/en/reference/property-packages-guide.html): criterios de selección por sistema.
- [DWSIM — Property Package Selection](https://dwsim.org/wiki/index.php?title=Property_Package_Selection): importancia de parámetros binarios y comparación experimental.
- [DWSIM — referencia de API y paquetes disponibles](https://dwsim.org/help/api/index.html).
- [ΣIM — hoja de ruta](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md) y [ejemplos](https://sim.lemeit.ar/ejemplos.html).

Las cifras de las actividades son referencias previamente documentadas en ΣIM, no ejecuciones nuevas ni mediciones experimentales. El contenido es una explicación propia y no reproduce íntegramente documentación externa.

**Próximo capítulo:** equilibrio líquido–vapor, fracción de vapor y lectura de los resultados flash.
