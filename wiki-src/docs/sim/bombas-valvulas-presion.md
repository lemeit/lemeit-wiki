# Bombas, válvulas y cambios de presión

**Capítulo 6 · Fundamentos de ΣIM.** [Capítulo anterior](energia-operaciones-termicas.md) · [Ejemplo 12](https://sim.lemeit.ar/ejemplos.html).

## 1. Presión absoluta y presión manométrica

El simulador solicita **presiones absolutas** en kPa. La conversión de referencia es: **p_abs = p_man + p_atm**. Una presión manométrica de cero no significa presión absoluta cero. Para pasar de kPa a bar: **1 bar = 100 kPa**.

Esta conversión merece atención especial: el adaptador MCP de ΣIM utiliza una interfaz que requiere bar en la prescripción de presión del equipo. En la implementación ya se corrigió la conversión **500 kPa → 5 bar** y **200 kPa → 2 bar**; no debe hacerse una conversión manual adicional al escribir la presión del inspector.

## 2. Bomba: presión y trabajo

Una bomba aumenta la presión del líquido mediante trabajo. En el modo actual de ΣIM se especifican **presión absoluta de descarga** y **eficiencia porcentual**. El trabajo y las propiedades de salida dependen del modelo termodinámico y el cálculo real.

Para una aproximación de líquido prácticamente incompresible, la potencia hidráulica ideal se estima como **V̇ Δp**. La potencia al eje requerida será aproximadamente **V̇ Δp/η**, si η representa la eficiencia de bomba correspondiente y no se incorporan otros términos. No asumir esta simplificación si hay cambios significativos de densidad, fase o propiedades.

## 3. Válvula: estrangulamiento

Una válvula reductora introduce caída de presión sin proporcionar trabajo útil de eje. Bajo las hipótesis habituales de estrangulamiento estacionario y adiabático, **hₑ ≈ hₛ**. Esto **no significa** que la temperatura necesariamente sea constante: a entalpía aproximadamente constante, la temperatura puede cambiar según fluido, presión y fases.

En ΣIM la válvula se define por la **presión absoluta de salida**. La dirección del cambio de presión y la plausibilidad del resultado deben revisarse en el cálculo, porque una alimentación intermedia puede no traer su presión explícita en el archivo JSON.

## 4. Práctica con el ejemplo 12

**Caso documentado:** agua a 25 °C y 1 kg/s, desde **101,325 kPa** hacia una bomba de **500 kPa** con **75 %** de eficiencia, seguida de una válvula con presión de salida de **200 kPa**.

1. Antes de ejecutar, anticipá cuál equipo requiere trabajo y cuál disipa presión.
2. Identificá dónde se especifican las presiones, y cuáles aparecen solo como resultados.
3. Verificá que el caudal másico se conserve en ausencia de fugas y reacciones.
4. Comprobá la entalpía aproximadamente constante a través de la válvula.
5. Comprobá 500 kPa absolutos después de la bomba y 200 kPa después de la válvula. Consultá la potencia calculada de la bomba y contrastala con ṁ(hsalida − hentrada); en la válvula, comprobá que la entalpía se conserva.

## 5. Errores habituales

- Confundir presión absoluta con manométrica.
- Especificar la misma presión de salida para bomba y válvula sin considerar el recorrido del proceso.
- Afirmar que un estrangulamiento es isotérmico.
- Calcular trabajo real con fórmulas de líquido incompresible sin comprobar que esa hipótesis sea adecuada.
- Interpretar un estado convergente como garantía de selección física correcta del equipo.

**Referencia técnica de alcance:** [hoja de ruta de ΣIM, 0.11.0](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md). Este capítulo no agrega modos nuevos de cálculo.
