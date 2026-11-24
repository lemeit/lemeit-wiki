# Diagnóstico, validación y confiabilidad de resultados

**Capítulo 10 · Fundamentos de ΣIM.** [Capítulo anterior](unidades-bases-conversiones.md) · [Comprobaciones del proyecto](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md).

## 1. Cuatro niveles diferentes

Una simulación puede superar una comprobación y fallar otra:

| Nivel | Pregunta | Ejemplo |
|---|---|---|
| Estructural | ¿Las corrientes y los puertos están conectados correctamente? | No hay entradas duplicadas ni lazos incompatibles |
| Especificaciones | ¿Los datos requeridos son suficientes y admitidos? | Presión, temperatura y composición definidas |
| Numérico | ¿El motor logró resolver con residuos aceptables? | Balance de masa dentro de tolerancia |
| Físico | ¿El modelo representa razonablemente la realidad? | Fases y entalpías contrastadas con datos fiables |

Que el motor finalice sin error es evidencia del nivel **numérico**, pero no constituye validación experimental del modelo físico.

## 2. Residuos de balances

Para materia sin reacción, el **residuo absoluto** es rₘ = Σṁ_entradas − Σṁ_salidas. Una medida relativa orientativa es |rₘ| dividido por una escala de caudal pertinente, por ejemplo el caudal total de entrada si es no nulo. La tolerancia debe declararse y justificarse: no hay un único umbral universal.

En reactores, para un componente *i* hay que considerar el término de generación/consumo. Un residuo de componente calculado sin reacción puede ser grande aunque la química esté bien representada; verificá entonces conservación de **elementos** y masa total.

Para energía, deben incluirse las corrientes térmicas y el trabajo asociados a la frontera elegida, además de los efectos de reacción según la formulación usada.

## 3. Ejemplo resuelto: cierre material del flash

Usando cifras históricas del Ejemplo 11: entrada **1,000000000 kg/s**, vapor **0,638235345 kg/s** y líquido **0,361757524 kg/s**. La suma de salidas es **0,999992869 kg/s** y el residuo **0,000007131 kg/s**. El residuo relativo a 1 kg/s es aproximadamente **7,131 × 10⁻⁶** (o **0,0007131 %**). Estos números proceden de valores redondeados; antes de declarar aprobación debe revisarse el criterio del [informe de flash](https://github.com/lemeit/lemeit-sim/blob/main/docs/flash-0.10.0.md).

Este ejemplo muestra por qué un valor no exactamente cero no prueba automáticamente un fallo: hay que analizar redondeo, método numérico y tolerancia elegida.

## 4. Diagnóstico de ΣIM

**Herramientas → Diagnóstico del proyecto** permite revisar errores básicos de alimentación, especificaciones y conexiones. El gateway comprueba adicionalmente tipos de puertos, índices, duplicidades, componentes, límites de parámetros y restricciones sobre las recirculaciones admitidas.

**Herramientas → Estado de motores** sirve para conocer disponibilidad y versión observada de Motor principal y Motor alternativo. No equivale a un diagnóstico termodinámico ni permite iniciar o reiniciar remotamente los equipos. La elección de motor es **manual y sin conmutación automática**.

## 5. Cómo documentar un ensayo reproducible

Conservá como mínimo: fecha, versión de ΣIM y DWSIM, motor Windows/Debian, componentes, paquete de propiedades, unidades, especificaciones de todas las alimentaciones, parámetros de cada equipo, proyecto JSON, resultados crudos, residuos/tolerancias y observaciones. Si comparás dos motores DWSIM, estás validando consistencia entre implementaciones, **no** independencia física del modelo.

## 6. Actividad de auditoría

Tomá un caso ya documentado; anotá qué se conocía **antes** de calcular y qué se obtuvo **después**. Verificá al menos conservación de materia y una magnitud energética. Cambiá deliberadamente un dato de entrada (sin perder la copia original) y compará los resultados. Si el diagnóstico informa una especificación ausente, corregila antes de interpretar cualquier resultado del motor.

## 7. Errores frecuentes

- Confundir «servicio disponible» con «cálculo válido».
- Ocultar residuos para que la tabla parezca cerrar exactamente.
- Comparar balances en unidades o bases diferentes.
- Considerar validación experimental la coincidencia entre dos rutas que usan el mismo DWSIM.
- Cambiar varias variables a la vez y luego atribuir la diferencia a una sola causa.

**Referencias:** [DWSIM — Common Mistakes](https://dwsim.org/tutorials/en/fundamentals/06-common-mistakes.html) · [CHECKS de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md). Guía original, sin ejecutar ensayos nuevos.
