# Taller de prácticas integradoras y ruta de aprendizaje

**Capítulo 11 · Fundamentos de ΣIM.** [Capítulo anterior](diagnostico-validacion.md) · [Biblioteca de ejemplos](https://sim.lemeit.ar/ejemplos.html).

## 1. Método de trabajo de un ingeniero

Para cada problema, seguí una secuencia trazable: **plantear → definir frontera → listar componentes → seleccionar modelo → contar especificaciones → construir el diagrama → calcular → verificar balances → interpretar físicamente → documentar**.

Evitar comenzar directamente por el botón «Calcular» ayuda a descubrir errores conceptuales. Un buen informe separa datos ingresados, ecuaciones, predicciones y comprobaciones.

## 2. Propuesta de recorrido de ocho actividades

| Actividad | Ejemplo de ΣIM | Contenidos | Producto de la práctica |
|---|---|---|---|
| 1 | 01 · Mezcla de agua | Masa y entalpía | Balance y predicción de temperatura |
| 2 | 04 · División | Especificaciones y GL | Caudales 70/30 y cierre |
| 3 | 02–03 · Calentamiento/enfriamiento | Primera ley, signos | Dos balances energéticos |
| 4 | 05 · Agua–etanol | Bases de composición, NRTL | Balance de componentes |
| 5 | 11 · Flash | Equilibrio y fases | Tabla de vapor/líquido, residuo |
| 6 | 12 · Bomba y válvula | Presión y trabajo | Perfil de presiones |
| 7 | 13 · Intercambiador | Transferencia de calor | Dos balances sin mezcla |
| 8 | 07–08 · Reactores | Estequiometría y energía | Balance de elementos |

Estas ocho actividades utilizan casos introductorios de la biblioteca; los ejercicios de recirculación posteriores requieren verificación específica y no se dan por validados por esta guía.

## 3. Práctica resuelta: mezcla y división

Supongamos dos alimentaciones de agua de **1 kg/s cada una**, a 20 °C y 80 °C, que entran en un mezclador adiabático. El flujo total resultante es **2 kg/s**. Si a continuación se incorpora un divisor con fracción **0,65** hacia la salida 1:

- Primera salida: **2 × 0,65 = 1,30 kg/s**.
- Segunda salida: **2 × 0,35 = 0,70 kg/s**.
- Cierre: **1,30 + 0,70 = 2,00 kg/s**.

Para calcular la temperatura real de mezcla con DWSIM, elegí un paquete pertinente (para agua pura, IAPWS-IF97) y no fijes arbitrariamente la temperatura de salida. El divisor modifica caudales, no representa por sí mismo un intercambiador de calor. **Este ejemplo combinado se formula como ejercicio; los caudales anteriores son balances analíticos, no una nueva corrida de motor.**

## 4. Práctica propuesta: intercambiador y separación

Diseñá sobre papel un proceso donde una corriente se calienta y posteriormente ingresa a un flash. Antes de reproducirlo en ΣIM, indicá: qué datos hay que fijar, qué paquete usar para agua–etanol, qué unidades intervienen, qué resultados salen del calentador y cuáles del flash, y qué balances verificarías.

**No inventes valores** de fracciones de vapor ni de temperaturas sin una corrida y sin conocer el paquete y las condiciones reales.

## 5. Rúbrica de evaluación sugerida

| Criterio | Puntos |
|---|---:|
| Planteo, frontera y variables | 20 |
| Especificaciones y selección termodinámica | 20 |
| Conexiones y coherencia del diagrama | 15 |
| Balances con unidades y residuos | 25 |
| Interpretación, límites y fuentes | 20 |
| **Total** | **100** |

La calificación no debe basarse únicamente en que el motor termine el cálculo. Un estudiante puede obtener un número físicamente inverosímil después de configurar mal un modelo.

## 6. Modelo breve de entrega

**Objetivo y esquema.** Identificar qué operación o fenómeno se estudia.

**Datos y supuestos.** Tabla con componentes, composición, modelo, presión, temperatura, caudales y parámetros de equipo.

**Cálculos previos.** Uno o dos balances analíticos y una predicción de orden de magnitud.

**Simulación.** Motor elegido, versión, archivo de proyecto y tabla de resultados.

**Verificación.** Residuos, diferencias respecto de la predicción, unidades y tolerancias.

**Conclusión.** Alcance físico de los resultados y próximos ensayos; distinguir siempre evidencia computacional de mediciones.

## 7. Próximos temas de nivel intermedio

Cuando ΣIM incorpore y valide los módulos correspondientes, la serie podrá crecer con reciclados y convergencia iterativa, columnas de separación, cinética y dimensionamiento de reactores. **Hoy son contenidos conceptuales o planificados, no operaciones disponibles en el editor.**

**Fuentes:** [DWSIM Tutorials](https://dwsim.org/tutorials/) · [roadmap de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/docs/roadmap.md). Actividades y secuencia redactadas específicamente para esta wiki.
