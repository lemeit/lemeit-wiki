# Unidades, bases de composición y conversiones

**Capítulo 9 · Fundamentos de ΣIM.** [Capítulo anterior](diagramas-conexiones.md) · [Guía de unidades](https://github.com/lemeit/lemeit-sim/blob/main/docs/unidades-y-calculadora.md).

## 1. Una magnitud, varias unidades

Los resultados solo son interpretables si conservan su **dimensión física**. ΣIM permite utilizar unidades de diferentes sistemas de ingeniería, pero la selección visual no debe confundirse con un cambio del proceso físico.

| Magnitud | Ejemplos de unidades | Conversión |
|---|---|---|
| Caudal másico | kg/s, kg/h | 1 kg/s = 3600 kg/h |
| Presión absoluta | kPa, bar | 100 kPa = 1 bar |
| Temperatura | °C, K | T(K) = T(°C) + 273,15 |
| Potencia | kW, kJ/s | 1 kW = 1 kJ/s |
| Entalpía específica | kJ/kg | Debe respetar su base y referencia |

Una **diferencia** de temperaturas de 10 °C equivale a 10 K; una temperatura absoluta de 10 °C **no** equivale a 10 K.

## 2. Caudal másico frente a caudal molar

La relación es **ṅ = ṁ / M**, si la masa molar *M* se expresa en unidades compatibles. Para una mezcla con fracciones molares *xᵢ*, la masa molar promedio es **M̄ = Σ xᵢ Mᵢ**.

Las fracciones másicas se relacionan con las molares mediante **wᵢ = xᵢ Mᵢ / ΣxⱼMⱼ**, y la inversa es **xᵢ = (wᵢ/Mᵢ) / Σ(wⱼ/Mⱼ)**. No se puede convertir correctamente una mezcla usando una masa molar arbitraria o sin definir la composición.

## 3. Ejemplo resuelto: agua y etanol

Tomemos una mezcla idealizada **solo para convertir composiciones**, con 1 mol de agua (M ≈ 18,015 g/mol) y 1 mol de etanol (M ≈ 46,069 g/mol). Por definición las fracciones molares son 0,5 y 0,5. Las masas correspondientes son 18,015 g y 46,069 g.

Por tanto, la fracción **másica de agua** es 18,015 / (18,015 + 46,069) ≈ **0,2811** y la de etanol ≈ **0,7189**. Una mezcla 50/50 **molar** no es una mezcla 50/50 **másica**. Este cálculo no requiere asumir equilibrio ideal; únicamente usa masas molares.

En los cálculos reales de ΣIM, las masas molares proceden del catálogo del motor.

## 4. Composición y cierres

Tanto las fracciones másicas como las molares suman 1 en su base respectiva. Para verificar un balance por componente deben usarse **caudales compatibles con las fracciones**: ṁᵢ = ṁwᵢ y ṅᵢ = ṅxᵢ. Multiplicar kg/s por fracción molar no entrega el caudal másico por componente salvo casos especiales.

## 5. Referencias de entalpía

El cero de entalpía depende de la convención y del paquete de propiedades. Es posible que dos programas informen diferentes valores absolutos de *h* y aun así coincidan en **diferencias de entalpía** para el mismo proceso y referencia coherente. No mezclar referencias de paquetes distintos dentro de un mismo balance sin justificación.

## 6. Ejercicios

1. Convertí **2,5 kg/s** a kg/h. **Respuesta: 9000 kg/h**.
2. Convertí **350 kPa** a bar. **Respuesta: 3,5 bar**.
3. Convertí **60 °C** a K. **Respuesta: 333,15 K**.
4. Una corriente de **2 kg/s** contiene 30 % másico de componente A. ¿Cuánto A transporta? **0,6 kg/s**.
5. Explicá por qué no se debe usar directamente 0,50 mol/mol como 50 % másico de agua–etanol.

**Referencia:** [DWSIM — Units and Numbers](https://dwsim.org/tutorials/en/fundamentals/05-units-and-numbers.html) y [guía de unidades de ΣIM](https://github.com/lemeit/lemeit-sim/blob/main/docs/unidades-y-calculadora.md). Ejercicios propios.
