# Grados de libertad y especificaciones en simulación de procesos

En un simulador de procesos, **no se trata de completar todos los campos de una pantalla**. Hay que elegir un conjunto suficiente y compatible de datos para que las ecuaciones del modelo determinen las incógnitas. Esta guía es una elaboración didáctica original, vinculada al [tutorial oficial de DWSIM sobre grados de libertad](https://dwsim.org/tutorials/en/fundamentals/02-degrees-of-freedom.html), y adaptada a las funciones disponibles en [ΣIM](https://sim.lemeit.ar/).

[Capítulo anterior: corrientes, estado y balances](fundamentos-corrientes-balances.md) · [Guía de los 13 ejemplos](https://sim.lemeit.ar/ejemplos.html).

## 1. ¿Qué son los grados de libertad?

En una formulación sencilla, los **grados de libertad (GL)** son la cantidad de variables independientes que todavía necesitamos especificar:

**GL = número de incógnitas independientes − número de ecuaciones independientes.**

Esta cuenta es *conceptual*, no una función implementada aún por el portal. Una ecuación repetida no aporta una restricción independiente; además, una solución algebraicamente determinada puede ser físicamente imposible.

| Resultado | Interpretación | Qué revisar |
|---|---|---|
| GL mayor que cero | Faltan especificaciones independientes | Condiciones de entrada, parámetros del equipo o conexiones |
| GL igual a cero | El sistema está formalmente determinado | Aun así hay que verificar consistencia, propiedades y convergencia |
| GL menor que cero | Hay más restricciones que incógnitas independientes | Variables que ya son resultados o especificaciones incompatibles |

**Ejemplo simple:** si conocemos el caudal de entrada y un divisor reparte una fracción *f* hacia la primera salida, la segunda queda fijada por el balance. Si se fijan arbitrariamente ambas salidas **y** una fracción que no coincide con ellas, las especificaciones resultan inconsistentes.

## 2. Corrientes de alimentación: ¿qué debo ingresar?

Para una corriente con composición conocida, habitualmente necesitamos definir el **caudal** y **dos propiedades intensivas independientes** para el estado termodinámico, además de escoger el modelo de propiedades. DWSIM ofrece distintas parejas de especificación flash, y ΣIM adapta las siguientes para alimentaciones:

| Modo | Datos intensivos que se especifican | Observación |
|---|---|---|
| TP | Temperatura y presión | Caso inicial más sencillo |
| PH | Presión y entalpía específica | La temperatura se calcula |
| PS | Presión y entropía específica | **No habilitado en ΣIM con IAPWS-IF97**, por una discrepancia comprobada del motor |
| PV | Presión y fracción molar de vapor | Requiere condiciones que admitan una solución física |
| TV | Temperatura y fracción molar de vapor | La presión se calcula |

Se indica **una única pareja de especificación activa**, no todas simultáneamente. Las fracciones de composición deben tener una base clara (másica o molar) y sumar uno. Las presiones que solicita ΣIM son **absolutas**, no manométricas.

Una **corriente calculada** a la salida de un equipo no debe tratarse como otra alimentación independiente: sus propiedades dependen de las ecuaciones y especificaciones del equipo anterior.

## 3. ¿Qué especifica cada operación en ΣIM?

La tabla resume el **contrato actual del editor**, no todo lo permitido por DWSIM de escritorio.

| Equipo | Datos o relaciones relevantes | Qué calcula el motor |
|---|---|---|
| Mezclador | Alimentaciones, composiciones y balance adiabático | Caudal, composición y estado de salida |
| Divisor | Fracción hacia la salida 1 | Caudales de ambas salidas |
| Calentador / enfriador | Temperatura objetivo de salida | Estado de salida y calor intercambiado |
| Bomba | Presión absoluta de salida y eficiencia | Estado de salida y trabajo asociado |
| Válvula | Presión absoluta de salida | Estado posterior al estrangulamiento |
| Intercambiador de dos corrientes | Potencia térmica conocida, positiva en el modo actual de ΣIM | Temperaturas de salida y balances de ambos lados |
| Separador flash | Alimentación y conexión de salidas de vapor y líquido | Distribución de fases según equilibrio |
| Reactor de conversión | Reacción, compuesto base, conversión, fase y temperatura de salida | Composición, energía y estados de salida |
| Reactor de equilibrio | Reacción, compuesto base, fase y temperatura de salida | Estado de equilibrio y salidas |

**Importante:** que DWSIM permita cambiar el modo de cálculo de un equipo no significa que ΣIM haya implementado todas esas variantes. Por ejemplo, en el intercambiador del portal **no se especifican a la vez potencia y temperatura de salida**; su modo didáctico verificado utiliza la potencia térmica. Los modos por UA, área o *rating* quedan pendientes.

## 4. Tres errores distintos que no debemos confundir

**A. Falta un dato (subespecificación).** Por ejemplo, agregar una bomba sin presión de salida o dejar sin conectar una corriente obligatoria. Acción: completar la propiedad o conexión que corresponde.

**B. Se imponen resultados como datos (sobreespecificación).** Por ejemplo, asignar una temperatura manual a la corriente calculada después de un calentador cuya temperatura de salida ya controla el equipo. Acción: identificar qué objeto determina cada variable y quitar la especificación redundante.

**C. Los datos no admiten una solución física (inconsistencia).** Por ejemplo, solicitar una fracción de vapor incompatible con las condiciones y el modelo, o imponer una presión físicamente inadecuada para el equipo. Acción: revisar restricciones físicas, fases, propiedades y rangos de validez. Un conteo formal GL = 0 **no garantiza** un resultado correcto.

DWSIM cuenta con herramientas más amplias de verificación (*Flowsheet Check* y *Explain Result*). **ΣIM todavía no reproduce íntegramente esas herramientas**, aunque sí valida conexiones, entradas y algunos parámetros antes de solicitar el cálculo.

## 5. Prácticas con los ejemplos de ΣIM

### Actividad A — Mezclador (Ejemplo 01)

1. Abrí [Ejemplo 01](https://sim.lemeit.ar/ejemplos.html) y distinguí dos alimentaciones de una salida calculada.
2. Verificá las dos temperaturas, presiones, caudales y composiciones de las entradas.
3. Predecí la masa total de salida mediante el balance: 1 + 1 = **2 kg/s**.
4. Calculá con Motor principal o Motor alternativo. La temperatura de referencia previamente validada es **50,021387 °C**.
5. Explicá por qué la temperatura de salida es un **resultado**, no un nuevo dato independiente.

### Actividad B — Divisor (Ejemplo 04)

1. Identificá la alimentación de **1 kg/s** y la fracción especificada de **0,70**.
2. Antes de calcular, determiná ambas salidas: **0,70 kg/s** y **0,30 kg/s**.
3. Respondé: si también impusiéramos 0,40 kg/s en la segunda salida, ¿cuál sería la contradicción?

### Actividad C — Calentador (Ejemplo 02)

1. Identificá una corriente de agua de entrada a **20 °C** y una temperatura objetivo de **60 °C**.
2. Anotá cuál temperatura es **dato** y cuál energía transferida es **resultado**.
3. Ejecutá el caso; la referencia de ΣIM es una potencia hacia el fluido de aproximadamente **+167,209679 kW**.
4. Compará con la descripción del calentador en DWSIM: su aplicación de escritorio también permite otros modos, pero estos deben implementarse y verificarse uno a uno en ΣIM.

### Actividad D — Intercambiador (Ejemplo 13)

El ejemplo de dos corrientes de agua trabaja con **40 kW especificados**, sin mezclar los fluidos. Compará las temperaturas calculadas de ambos lados con la guía del ejemplo. Explicá por qué sería incorrecto imponer simultáneamente ambos resultados de temperatura sin cambiar las ecuaciones o el modo de cálculo.

## 6. Lista de revisión antes de calcular

- ¿Todos los compuestos y el modelo termodinámico están elegidos?
- ¿Cada alimentación tiene caudal, composición y el par intensivo activo correspondiente?
- ¿La composición tiene una sola base identificada y sus fracciones suman uno?
- ¿Todos los puertos obligatorios están conectados en la dirección correcta?
- ¿Los parámetros del equipo corresponden al **modo de cálculo implementado**?
- ¿Se distinguen las salidas calculadas de los datos ingresados?
- ¿Las unidades son consistentes, especialmente presión absoluta y caudal?
- ¿Las condiciones son físicamente plausibles y están dentro del dominio del modelo?

**En ΣIM:** usá la revisión de especificaciones y conexiones antes de calcular; los mensajes orientan pero no constituyen todavía un análisis numérico completo de grados de libertad. Cuando falla una simulación, distinguí errores de entrada, de conexión y de propiedades termodinámicas.

## 7. Conteo de grados de libertad: tres casos resueltos

Para contar GL primero definimos la **frontera del sistema** y qué variables ya vienen determinadas por sus alimentaciones. Los siguientes conteos son modelos didácticos reducidos; no representan el número total de variables internas que maneja el solucionador termodinámico.

### Caso 1 — Divisor sin reacción

Una alimentación de agua de 1 kg/s se distribuye en dos productos. Las incógnitas son los caudales de salida \(m_1\) y \(m_2\). Solo contamos con el balance global:

\[
m_1+m_2=1\ \mathrm{kg/s}
\]

Por tanto: **2 incógnitas − 1 ecuación = 1 GL**. Necesitamos una especificación adicional. Si elegimos la fracción de reparto \(f=0{,}70\), queda \(m_1=f\cdot 1=0{,}70\ \mathrm{kg/s}\) y \(m_2=0{,}30\ \mathrm{kg/s}\), con **GL = 0**. No corresponde fijar libremente un tercer dato independiente, por ejemplo \(m_2=0{,}40\ \mathrm{kg/s}\), porque sería incompatible con los anteriores.

En ΣIM la fracción es el dato del divisor; ambos caudales de producto son resultados.

### Caso 2 — Mezclador adiabático de agua

Consideremos dos alimentaciones conocidas de 1 kg/s a 20 °C y 80 °C, con presión y composición definidas, y una sola salida. En una aproximación monofásica de capacidad calorífica constante, las incógnitas de este subconjunto son \(m_s\) y \(T_s\). Hay dos ecuaciones independientes: balance de masa y balance de energía. Por ello **GL = 2 − 2 = 0**, sin fijar la temperatura de salida.

\[
m_s=m_a+m_b=2\ \mathrm{kg/s}
\]

\[
m_sc_pT_s=m_ac_pT_a+m_bc_pT_b
\]

La estimación simplificada es \(T_s=50\ ^\circ\mathrm C\). En el Ejemplo 01 DWSIM calcula **50,021387 °C** con su modelo de propiedades: esa diferencia no es un error, sino una consecuencia de usar entalpías termodinámicas en lugar de \(c_p\) constante.

### Caso 3 — Calentador con temperatura objetivo

Con una alimentación de 1 kg/s a 20 °C y propiedades conocidas, tomemos como incógnitas la temperatura de salida \(T_s\) y la potencia térmica \(Q\). El balance de energía proporciona **una ecuación para dos incógnitas**: inicialmente **GL = 1**. En el modo implementado por ΣIM se especifica \(T_s=60\ ^\circ\mathrm C\); la potencia pasa a ser resultado y el modelo queda con **GL = 0**.

En el Ejemplo 02, DWSIM obtiene aproximadamente **+167,209679 kW**. Si se especificara también ese valor como dato independiente, se impondría una restricción adicional que el modo actual del equipo no necesita.

## 8. Qué valida realmente el diagnóstico de ΣIM

La comprobación previa del editor **no calcula automáticamente GL**. Detecta condiciones de alimentación inválidas, parámetros faltantes o fuera de rango y conexiones materiales incompletas. El gateway `server.py` sigue siendo la autoridad: además de los valores numéricos, rechaza puertos ocupados dos veces, conexiones entre objetos incompatibles, índices incorrectos, ciclos de recirculación y otras estructuras no admitidas.

| Comprobación | Diagnóstico del editor | Gateway |
|---|---|---|
| Alimentaciones: par flash, caudal y composición | Revisión básica | Validación obligatoria |
| Cantidad de entradas/salidas materiales por equipo | Avisos antes del cálculo | Validación obligatoria |
| Tipos de puertos, índices y ocupación | Se controla al conectar | Se valida nuevamente al recibir el proceso |
| Ciclos/recirculaciones | No hay análisis algebraico completo | Rechaza ciclos |
| Consistencia física, equilibrio y convergencia | No garantizada | Depende del cálculo real de DWSIM |

**Criterio de uso:** un diagnóstico sin advertencias significa solamente que **no se detectaron errores básicos**. No demuestra GL = 0, factibilidad física ni convergencia. Si el motor rechaza el proceso, conservar el mensaje original es parte del análisis.

## 9. Lecturas y alcance

- [DWSIM — Degrees of Freedom](https://dwsim.org/tutorials/en/fundamentals/02-degrees-of-freedom.html), tutorial oficial (en inglés), utilizado como referencia conceptual.
- [DWSIM — Streams](https://dwsim.org/help/main/23-streams.html), guía oficial de modos flash, composición y corrientes.
- [DWSIM — Streams, State and Balances](https://dwsim.org/tutorials/en/fundamentals/01-streams-state-and-balances.html), fundamento del capítulo anterior.
- [ΣIM — catálogo](https://sim.lemeit.ar/catalogo.html) y [roadmap](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md): alcance real del portal y funciones previstas.

La redacción, los ejemplos y la secuencia didáctica de esta página son propios. No es una traducción oficial ni supone permiso de reproducción íntegra de los tutoriales de terceros. Las cifras son **resultados de modelos previamente documentados**, no mediciones experimentales nuevas.

**Próximo capítulo:** elección de modelos termodinámicos y criterios de validez para agua pura, mezclas no ideales y sistemas de hidrocarburos.
