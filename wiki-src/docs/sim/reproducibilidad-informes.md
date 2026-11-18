# Reproducibilidad, trazabilidad y presentación de informes

**Nivel intermedio · Capítulo 16.** Un resultado numérico aislado no permite que otra persona repita el estudio. Este capítulo propone una forma breve de documentar los proyectos creados en ΣIM y sus límites de validación.

[Diagnóstico y validación](diagnostico-validacion.md) · [Prácticas integradoras](practicas-integradoras.md)

## 1. Distinguir cuatro niveles de evidencia

| Nivel | Evidencia | Qué puede afirmarse |
|---|---|---|
| Planteo analítico | Ecuaciones y unidades comprobadas | El modelo simplificado cierra bajo sus hipótesis |
| Prueba automática | Tests de código o contrato | El software responde según condiciones verificadas |
| Cálculo DWSIM | Ejecución identificada, versión y datos | El motor resuelve ese caso numérico |
| Validación experimental | Comparación con mediciones independientes | El modelo describe el fenómeno dentro del error analizado |

**Dos gateways que invocan DWSIM no constituyen dos modelos físicos independientes.** Su coincidencia aporta evidencia de consistencia de implementación, no prueba experimental.

## 2. Ficha mínima de simulación

Registrar al menos:

1. Objetivo, frontera del sistema e hipótesis.
2. Identificación del proyecto, diagrama y componentes.
3. Paquete termodinámico y razón de selección.
4. Entradas: unidades, bases, composición, condiciones y parámetros de equipos.
5. Motor seleccionado (Motor principal o Motor alternativo), versión del portal y del gateway observada, fecha y hora.
6. Salidas principales, balances y residuos con tolerancias elegidas.
7. Advertencias, limitaciones y próxima comprobación necesaria.

En ΣIM se pueden guardar archivos JSON de proyecto y exportar resultados según las funciones vigentes. No insertar claves de acceso, tokens, rutas privadas ni credenciales en informes públicos.

## 3. Ejemplo de informe: mezcla de agua

**Objetivo:** comparar el balance de materia y energía de una mezcla estacionaria de agua.

**Entradas:** dos alimentaciones de 1 kg/s, una a 20 °C y otra a 80 °C, ambas a 101,325 kPa absolutos; IAPWS-IF97, agua pura.

**Predicción analítica:** flujo de salida 2 kg/s. Con \(c_p\) constante e igual para ambas alimentaciones, temperatura aproximada 50 °C.

**Referencia ya documentada en ΣIM:** caudal de salida 2 kg/s y temperatura calculada 50,021387 °C. **No es una corrida nueva de esta edición de la wiki.**

**Comprobación pendiente si se repite el ensayo:** registrar el motor, las entalpías en la misma base y los residuos numéricos, además de la identificación del paquete y la versión real.

## 4. Interpretación de residuos

En una operación sin reacción, un residuo global de materia puede definirse como:

\[
\varepsilon_m=\sum\dot m_{in}-\sum\dot m_{out}
\]

Conviene informar el residuo con unidad (kg/s), y si se usa el error relativo especificar el denominador. Una diferencia no nula pequeña puede responder a redondeo o tolerancias; una grande debe investigarse.

En reacciones, comprobar también la **conservación elemental**: los caudales de una especie reaccionante no necesariamente se conservan.

## 5. Actividad final de auditoría

Elegí entre los ejemplos **01, 05, 11, 12 o 13** de la biblioteca. Prepará un informe de una o dos páginas con la ficha mínima, una tabla de tres entradas significativas, una tabla de tres resultados, el balance global y al menos dos limitaciones. Adjuntá el JSON para repetir el cálculo.

**Rúbrica sugerida (10 puntos):**

| Criterio | Puntos |
|---|---:|
| Problema, hipótesis y elección del modelo | 2 |
| Datos, unidades y trazabilidad | 2 |
| Resultados y balances | 3 |
| Interpretación y límites | 2 |
| Claridad y posibilidad de reproducción | 1 |

## 6. Alcance de la documentación

La serie de la wiki es original y tiene finalidad educativa. No reemplaza manuales oficiales de DWSIM, análisis de seguridad de procesos, diseño mecánico, regulación industrial ni contrastación experimental.

**Referencias:** [Documentación técnica de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/docs/README.md) · [Evidencias y comprobaciones](https://github.com/lemeit/lemeit-sim/blob/main/CHECKS.md) · [DWSIM](https://dwsim.org/tutorials/).
