# Balance de energía y operaciones térmicas

**Capítulo 5 · Fundamentos de ΣIM.** [Capítulo anterior](equilibrio-fases-flash.md) · [Ejemplos](https://sim.lemeit.ar/ejemplos.html).

## 1. Primera ley en régimen estacionario

Para un volumen de control estacionario, con una convención consistente de signos:

**Q̇ − Ẇₛ + Σ ṁₑ hₑ = Σ ṁₛ hₛ**, si se desprecian los cambios de energía cinética y potencial y Ẇₛ representa trabajo de eje **saliente** del volumen de control. El calor Q̇ es positivo cuando entra al sistema.

El uso de entalpía *h* (kJ/kg) incorpora energía interna y trabajo de flujo. Multiplicar kg/s por kJ/kg da kJ/s = **kW**. Para equipos con reacción, hay que verificar cómo el motor incorpora el calor de reacción y las referencias de entalpía; no reducir el problema arbitrariamente a «calor sensible».

## 2. Mezcla adiabática

Para dos alimentaciones y una salida sin trabajo ni transferencia térmica externa: **ṁ₁h₁ + ṁ₂h₂ = ṁₛhₛ**, junto a **ṁₛ = ṁ₁ + ṁ₂**. La temperatura de la salida se obtiene de la entalpía y el modelo seleccionado, no necesariamente del promedio aritmético de temperaturas.

En el **Ejemplo 01**, 1 kg/s de agua a 20 °C y 1 kg/s a 80 °C producen 2 kg/s a **50,021387 °C**, según el resultado histórico de DWSIM con IAPWS-IF97.

## 3. Calentador y enfriador

Si el equipo no desarrolla trabajo de eje y los cambios cinéticos/potenciales son despreciables: **Q̇ = ṁ(hₛ − hₑ)**. En ΣIM, el modo de calentador y enfriador expone como **especificación** la temperatura objetivo de salida; la potencia térmica es **resultado**.

| Caso en ΣIM | Alimentación | Salida objetivo | Potencia registrada |
|---|---|---|---|
| Ejemplo 02: calentador | 1 kg/s de agua, 20 °C | 60 °C | **+167,209679 kW** |
| Ejemplo 03: enfriador | 1 kg/s de agua, 80 °C | 30 °C | **−209,157884 kW** |

Estas cifras son referencias previamente calculadas, no valores medidos. Que un equipo se llame «enfriador» no justifica cambiar el signo calculado por conveniencia: hay que respetar el convenio explícito de calor hacia el fluido.

## 4. Intercambiador de dos corrientes

Con pérdidas térmicas externas despreciables y sin mezcla, el calor cedido por la corriente caliente debe igualar aproximadamente el absorbido por la fría, en valor absoluto. **No se suman los caudales de ambos lados**, porque los fluidos no se mezclan.

El **Ejemplo 13** especifica **40 kW transferidos del lado caliente al frío**. A 1 kg/s de agua por lado y 300 kPa, el motor obtiene aproximadamente **80 → 70,456556 °C** (caliente) y **20 → 29,565765 °C** (fría). La discrepancia térmica entre lados registrada fue de unos **0,00000033 kW**. ΣIM no habilita en este modo cálculos de diseño por UA o área.

### Distinguir transferencia interna del cierre energético

En el intercambiador del Ejemplo 13, **40 kW** circulan entre los dos lados, pero la suma algebraica de los cambios de entalpía de ambos lados es aproximadamente **cero**. Ese residuo no debe rotularse como potencia intercambiada: un balance global adiabático puede cerrar y al mismo tiempo existir una transferencia de calor elevada entre fluidos.

Para cada lado, `ΔḢ = ṁ_s h_s − ṁ_e h_e`, en kW. El lado caliente tiene `ΔḢ < 0` y el frío `ΔḢ > 0`; el residuo es la suma. Distinguir las magnitudes individuales del valor neto informado por el gateway. La presentación ampliada en paneles y exportaciones está propuesta para [0.47.5](https://github.com/lemeit/lemeit-sim/pull/135).

## 5. Ejercicio resuelto independiente

Una corriente de 2 kg/s de líquido con **cₚ constante = 4 kJ/(kg·K)** se calienta 15 K sin cambios de fase. Entonces Q̇ ≈ 2 × 4 × 15 = **120 kW**. Este ejercicio es una aproximación ideal pedagógica: para comparar con DWSIM deben utilizarse las entalpías del modelo y comprobar el estado de fases.

## 6. Práctica y revisión

Compará Q̇ calculado desde las entalpías de entrada/salida en los ejemplos 02 y 03. Revisá unidades, signos y condiciones antes de llamar «error» a una diferencia. En el intercambiador, verificá por separado los balances de masa y la ausencia de mezcla de composiciones.

**Fuente:** [DWSIM — Tutorials](https://dwsim.org/tutorials/), secciones de calentadores, mezcladores e intercambiadores. Guía original, con resultados históricos documentados en [ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/README.md).
