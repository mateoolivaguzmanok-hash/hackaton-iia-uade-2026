# M06 · ¿Rotamos a este jugador?

> Hackatón IIA · UADE · 3 de noviembre de 2026

**Integrantes:** Mateo Oliva · Ángeles Grinceri  
**IA declarada:** ChatGPT  
**Pista elegida:** Sistema de reglas SI–ENTONCES  
**Publicación en LinkedIn:** Pendiente

## 1. Formulación y PEAS

### La pregunta

El cuerpo técnico de un club de fútbol de Primera División necesita identificar jugadores que podrían sufrir una lesión muscular para evaluar si conviene darles descanso en la próxima fecha.

**Pregunta binaria:** ¿El jugador sufrirá una lesión muscular durante los próximos 14 días?

- **SÍ (1):** El jugador sufrirá una lesión muscular.
- **NO (0):** El jugador no sufrirá una lesión muscular.

La predicción sirve como apoyo para que el cuerpo técnico y el preparador físico evalúen una posible rotación. El sistema no toma por sí solo la decisión definitiva.

### PEAS

| Elemento | Descripción |
|---|---|
| **P — Medida de rendimiento** | Detectar correctamente a los jugadores que podrían lesionarse, priorizando la sensibilidad y controlando las falsas alarmas. |
| **E — Entorno** | Club de fútbol profesional, jugadores, entrenamientos y preparación de la próxima fecha. |
| **A — Actuadores** | Emitir una predicción SÍ/NO y una alerta para que el cuerpo técnico evalúe la rotación. |
| **S — Sensores** | Minutos jugados en los últimos 7 días, distancia recorrida a alta intensidad y horas promedio de sueño. |

### Objetivo real y métrica

**Objetivo real:** reducir las lesiones musculares y sus consecuencias deportivas, evitando perder jugadores durante varias semanas.

**Métrica prioritaria:** sensibilidad (*recall*), porque necesitamos detectar a los jugadores que efectivamente sufrirán una lesión.

Sin embargo, también debemos controlar los falsos positivos, ya que rotar innecesariamente a muchos titulares perjudicaría al equipo.

La sensibilidad es un indicador del desempeño predictivo, pero no demuestra por sí sola que rotar jugadores reduzca las lesiones.

## 2. El sistema

### Cómo decide

Elegimos un **sistema de reglas SI–ENTONCES** porque es sencillo, transparente, reproducible y permite explicar cada decisión.

El sistema utiliza tres variables disponibles antes de la próxima fecha:

1. **Minutos jugados:** representan parte de la carga física reciente.
2. **Distancia a alta intensidad:** representa el esfuerzo físico de alta exigencia.
3. **Sueño promedio:** funciona como una posible señal de recuperación.

### Regla de decisión

**SI:**

- Minutos jugados ≥ 150.
- Distancia a alta intensidad ≥ 3,4 km.
- Sueño promedio ≤ 7,5 horas.

**ENTONCES:** predecir **SÍ (1)**.

**EN CASO CONTRARIO:** predecir **NO (0)**.

Las tres condiciones deben cumplirse simultáneamente.

Los umbrales se eligieron observando exclusivamente los 14 casos de entrenamiento. No son valores médicos validados y no se modificaron después de observar los resultados de prueba.

### Variables utilizadas y descartadas

Se utilizan las tres variables disponibles porque representan diferentes aspectos de la carga física y la recuperación.

No utilizamos el resultado real como entrada del sistema, ya que solamente se conoce después de los 14 días y utilizarlo produciría una filtración de información.

Tampoco utilizamos el número de caso ni la pertenencia a entrenamiento o prueba como variables predictoras.

### Cómo se ejecuta en vivo

1. Recibir los minutos jugados, la distancia a alta intensidad y el sueño promedio de un jugador.
2. Comparar los tres valores con los umbrales establecidos.
3. Si cumple las tres condiciones, devolver SÍ (1).
4. En cualquier otro caso, devolver NO (0).
5. Comunicar el resultado al cuerpo técnico.

El sistema puede ejecutarse manualmente o mediante el siguiente código Python:

```python
def predecir_lesion(minutos, intensidad, sueno):
    if minutos >= 150 and intensidad >= 3.4 and sueno <= 7.5:
        return 1
    else:
        return 0

# Ejemplo
resultado = predecir_lesion(200, 3.6, 6.5)
print("Predicción:", resultado)
```

### Dos casos de ejemplo

| Caso inventado | Minutos | Intensidad | Sueño | Predicción |
|---|---:|---:|---:|---|
| Jugador A | 200 | 3,6 km | 6,5 h | SÍ (1) |
| Jugador B | 100 | 2,0 km | 8,0 h | NO (0) |

El jugador A cumple las tres condiciones, mientras que el jugador B no las cumple.

## 3. Medición

El conjunto contiene 20 casos sintéticos.

- **Entrenamiento:** casos 1 al 14.
- **Prueba:** casos 15 al 20.

La separación se respetó durante todo el desarrollo. El conjunto de prueba no se utilizó para elegir ni modificar los umbrales.

### Matriz de confusión — Entrenamiento

| Resultado real | Predice SÍ | Predice NO |
|---|---:|---:|
| **Real SÍ** | VP = 2 | FN = 0 |
| **Real NO** | FP = 0 | VN = 12 |

Total: 14 casos.

### Matriz de confusión — Prueba

| Resultado real | Predice SÍ | Predice NO |
|---|---:|---:|
| **Real SÍ** | VP = 0 | FN = 1 |
| **Real NO** | FP = 0 | VN = 5 |

Total: 6 casos.

### Métricas

**Exactitud (Accuracy)** = (VP + VN) / (VP + VN + FP + FN)

**Precisión (Precision)** = VP / (VP + FP)

**Sensibilidad (Recall)** = VP / (VP + FN)

**Especificidad** = VN / (VN + FP)

| Métrica | Entrenamiento | Prueba |
|---|---:|---:|
| Exactitud | 100 % | 83,33 % |
| Precisión | 100 % | No definida (0/0) |
| Sensibilidad | 100 % | 0 % |
| Especificidad | 100 % | 100 % |

La precisión en prueba no está matemáticamente definida porque el sistema no predijo ningún positivo. Algunas bibliotecas informáticas informan 0 % por convención, pero en este trabajo se explicita la división por cero.

### Entrenamiento contra prueba

El sistema clasificó correctamente los 14 casos de entrenamiento, pero falló al detectar la única lesión del conjunto de prueba.

En el caso 20, el jugador tenía:

- 343 minutos jugados.
- 3,0 km a alta intensidad.
- 5,7 horas de sueño.
- Resultado real: SÍ (1).

El sistema predijo NO porque no alcanzó el umbral de 3,4 km de alta intensidad.

Este resultado es compatible con un posible **sobreajuste (overfitting)**: la regla funciona perfectamente con los ejemplos utilizados para construirla, pero no logra generalizar adecuadamente para detectar lesiones en los nuevos casos.

El 100 % obtenido en entrenamiento no significa que el sistema tenga un 100 % de efectividad en situaciones reales.

## 4. El error más caro

Existen dos errores posibles:

**Falso positivo:** el sistema predice lesión, pero el jugador no se lesiona. El cuerpo técnico podría rotar innecesariamente a un titular, perjudicando el rendimiento del equipo.

**Falso negativo:** el sistema predice que no habrá lesión, pero el jugador termina lesionándose. El equipo podría perderlo durante tres o cuatro semanas.

Consideramos más costoso el falso negativo porque una lesión puede afectar varios partidos y la disponibilidad del jugador durante un período prolongado.

El sistema cometió exactamente un falso negativo en prueba, correspondiente al caso 20.

**Ajuste y mejora propuesta:** para una futura versión, estudiaríamos reglas menos restrictivas que permitan identificar más jugadores en riesgo, aunque esto pueda aumentar las falsas alarmas.

No modificamos los umbrales originales utilizando el caso 20 porque eso implicaría ajustar el sistema con los datos reservados para evaluación. Cualquier regla nueva debería desarrollarse con datos de entrenamiento y validarse posteriormente con nuevos casos independientes.

## 5. ¿Comparado contra qué?

### Modelo trivial: predecir siempre NO

Como la mayoría de los jugadores no se lesionan, utilizamos como referencia un sistema que siempre predice NO.

**Entrenamiento:**

- Aciertos: 12 de 14.
- Exactitud: 85,71 %.
- Sensibilidad: 0 %.
- Especificidad: 100 %.

**Prueba:**

- Aciertos: 5 de 6.
- Exactitud: 83,33 %.
- Sensibilidad: 0 %.
- Especificidad: 100 %.

### Comparación

| Indicador | Nuestra regla (prueba) | Siempre NO (prueba) |
|---|---:|---:|
| Exactitud | 83,33 % | 83,33 % |
| Sensibilidad | 0 % | 0 % |
| Especificidad | 100 % | 100 % |

Nuestra regla supera al modelo trivial en entrenamiento, pero **no lo supera en prueba**.

Esto demuestra que una exactitud aparentemente elevada no garantiza que el sistema cumpla su verdadero objetivo.

### Checklist de evaluación

**¿Qué datos tenemos?**  
20 casos sintéticos, con tres variables de entrada y una etiqueta binaria de lesión.

**¿Qué medimos?**  
Exactitud, precisión, sensibilidad y especificidad. Priorizamos la sensibilidad por el costo de los falsos negativos.

**¿Contra qué comparamos?**  
Contra el modelo trivial que siempre predice NO.

**¿Quién midió?**  
El equipo calculó las matrices y las métricas a partir de las predicciones de la regla, con apoyo de ChatGPT. Los resultados deben poder reproducirse manualmente durante la defensa.

## 6. Riesgo: el camino completo

### Datos → Modelo → Decisión → Consecuencia

**Datos:** minutos jugados, intensidad física y horas de sueño. Son indicadores parciales del estado del jugador y no incluyen antecedentes médicos, lesiones previas ni otros factores relevantes.

**Modelo:** reglas fijas construidas con solamente 14 casos de entrenamiento. Los umbrales podrían no representar adecuadamente la realidad.

**Decisión:** el sistema predice SÍ o NO y el cuerpo técnico utiliza esa información para evaluar una posible rotación.

**Consecuencia:** si el sistema produce un falso negativo, podría transmitir una falsa sensación de seguridad y el jugador podría continuar jugando pese a existir riesgo de lesión.

### Riesgo ético y operativo

El principal riesgo es confiar excesivamente en una predicción obtenida con pocos datos y convertirla automáticamente en una decisión deportiva.

Además, el sueño promedio podría no representar correctamente la recuperación física de todos los jugadores.

**Medida de prevención:** utilizar el sistema solamente como apoyo, mantener supervisión humana y no reemplazar la evaluación del preparador físico ni del personal médico.

Los datos son sintéticos. No se utilizaron registros médicos reales.

## 7. Bitácora

### Uso de la IA

**IA utilizada:** ChatGPT.

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Comprender la problemática M06 y formular el PEAS. | La definición del objetivo, las entradas disponibles y el costo de los errores. | Confundir la predicción de lesión con la decisión definitiva de rotación. |
| 2 | Construir un sistema de reglas SI–ENTONCES con los 14 casos de entrenamiento. | La regla de 150 minutos, 3,4 km y 7,5 horas de sueño como modelo inicial reproducible. | Considerar los umbrales como criterios médicos validados. |
| 3 | Evaluar el sistema sobre entrenamiento y prueba. | Las matrices de confusión y las métricas calculadas. | Interpretar el 100 % de entrenamiento como garantía de efectividad. |
| 4 | Analizar el error más costoso y comparar con una línea base. | La prioridad de la sensibilidad y la comparación con el clasificador siempre NO. | Modificar los umbrales utilizando el conjunto de prueba para aparentar un mejor resultado. |
| 5 | Redactar el README para su evaluación. | Una presentación reproducible con resultados, limitaciones y riesgos. | Inventar resultados de evaluación o afirmar que el sistema ya está validado para uso real. |

### Corridas contra el evaluador

| Corrida | Nota | Problema identificado | Cambio realizado |
|---|---|---|---|
| 1 | Pendiente | Pendiente de recibir la devolución del evaluador | Pendiente |

**Estado:** primera versión del README preparada para evaluación. Todavía no se registraron resultados del bot.

## Conclusión

Construimos un sistema de inteligencia artificial basado en reglas SI–ENTONCES para predecir lesiones musculares en jugadores de fútbol.

El sistema alcanzó un 100 % de exactitud en entrenamiento, pero solamente un 83,33 % en prueba, con sensibilidad del 0 %.

La conclusión principal es que **una exactitud elevada no significa necesariamente que un sistema sea útil**. Nuestro modelo no detectó la única lesión del conjunto de prueba y no superó al clasificador trivial que siempre predice NO.

Reconocemos esta limitación y proponemos como trabajo futuro mejorar la sensibilidad utilizando más datos de entrenamiento y validaciones independientes.

El sistema es un prototipo académico explicable y reproducible, no una herramienta médica validada.
