# Corte 3 – Machine Learning e integración final

> **Fecha límite de entrega: jueves 15 de octubre de 2026, 23:59.**

## 1. Integrantes

- Integrante 1: Jorge Luis Murillo Valverde (C35519)

## 2. Caso seleccionado

**Caso:** Caso 04 – Mesa de servicio de TI.

**Archivo original:** `datos/raw/caso_04_soporte_ti.csv`.

## 3. Hallazgo de Process Mining que motiva el modelo

En el análisis del Corte 2 se identificaron 51 variantes del proceso. La variante más frecuente fue la ruta directa `Incidente registrado → Clasificación → Asignación → Diagnóstico → Resolución → Validación → Cierre`, con 1.378 casos (55,12 %).

También se observó una asociación entre las reasignaciones y una mayor duración: los incidentes con al menos una reasignación antes de la primera Resolución tardaron en promedio 21,92 horas y tuvieron una tasa de incumplimiento del SLA del 31,70 %. Los casos sin reasignación tardaron en promedio 11,46 horas y tuvieron una tasa del 10,23 %.

Estos resultados motivan estudiar si la información conocida hasta el primer Diagnóstico puede ayudar a anticipar qué incidentes incumplirán el SLA. La asociación con reasignaciones no demuestra causalidad.

## 4. Problema de Machine Learning

### Pregunta

¿Podemos identificar, inmediatamente después del primer Diagnóstico, los incidentes que terminarán incumpliendo su SLA?

### Variable objetivo

`incumple_sla`:

- `1`: el tiempo calendario desde `Incidente registrado` hasta la primera `Resolución` supera `sla_horas`.
- `0`: el tiempo hasta la primera `Resolución` es menor o igual a `sla_horas`.

La definición sigue el diccionario de datos del caso. No se utiliza el último evento como final del SLA.

### Tipo de problema

- [x] Clasificación
- [ ] Regresión
- [ ] Agrupamiento
- [ ] Otro

### Justificación

La variable objetivo tiene dos categorías. La clasificación puede ayudar a priorizar la revisión de incidentes con riesgo de incumplimiento, sin sustituir la decisión de las personas responsables del soporte.

## 5. Variables predictoras

Las variables se construyen con información conocida al inicio del incidente o disponible hasta el primer Diagnóstico.

| Variable | Tipo | Origen | Justificación |
| --- | --- | --- | --- |
| `prioridad` | Categórica | CSV original, atributo del caso | Indica la urgencia y se relaciona con el SLA comprometido |
| `categoria` | Categórica | CSV original, atributo del caso | Describe el tipo de incidente |
| `canal` | Categórica | CSV original, atributo del caso | Permite comparar las vías de reporte |
| `departamento` | Categórica | CSV original, atributo del caso | Describe el departamento que reportó |
| `sla_horas` | Numérica | CSV original, atributo del caso | Es el límite comprometido conocido desde el inicio |
| `minutos_hasta_diagnostico` | Numérica derivada | Diferencia entre registro y primer Diagnóstico | Indica cuánto tiempo ha transcurrido hasta el momento de predicción |
| `eventos_hasta_diagnostico` | Numérica derivada | Conteo de eventos del prefijo | Resume cuántas actividades ocurrieron hasta ese momento |
| `reasignaciones_hasta_diagnostico` | Numérica derivada | Conteo de Reasignación en el prefijo | Captura transferencias conocidas hasta el punto de predicción |
| `escalamientos_hasta_diagnostico` | Numérica derivada | Conteo de Escalamiento en el prefijo | Captura escalaciones conocidas hasta ese momento |
| `recurso_en_diagnostico` | Categórica | Evento del primer Diagnóstico | Identifica el recurso registrado en ese punto |
| `grupo_en_diagnostico` | Categórica | Evento del primer Diagnóstico | Identifica el grupo asignado si ese dato está registrado |

Las variables derivadas del proceso se calculan solo con el prefijo observado hasta el primer Diagnóstico.

## 6. Preparación de datos

### Calidad y construcción de la etiqueta

El archivo original contiene 21.022 filas y 10 columnas. Se eliminaron 86 duplicados exactos en una copia en memoria, se identificaron 37 eventos con timestamp inválido y se excluyeron esos eventos del análisis temporal. Se estandarizaron las diferencias de mayúsculas, tildes y espacios en los nombres de actividad, unificando 38 formas de escritura en 12 actividades.

Con los eventos válidos se obtuvieron 2.488 incidentes con registro y primera Resolución temporales utilizables. En 390 se observó incumplimiento del SLA (15,68 %). Para el modelo se utilizaron 2.481 casos: se excluyeron 7 de la cohorte de modelado porque no tenían un primer Diagnóstico válido anterior o igual a la primera Resolución. Los 2.481 casos incluyen 390 incumplimientos (15,72 %).

Los eventos originales de `datos/raw/` no se modifican. El notebook realiza las transformaciones de forma reproducible.

### Momento de predicción

La predicción se realiza inmediatamente después del primer evento `Diagnóstico`. Solo se utilizan atributos del caso conocidos hasta ese punto y eventos anteriores o iguales a dicho Diagnóstico.

### Variables descartadas por fuga de información

No se utilizan como predictores:

- la duración hasta la primera Resolución;
- la hora de la primera Resolución o cualquier evento posterior al punto de predicción;
- la duración total hasta el último evento;
- el número final de reasignaciones o escalamientos;
- `incumple_sla`, porque es la variable objetivo.

La duración hasta la primera Resolución se usa exclusivamente para crear la etiqueta. `sla_horas` sí es admisible como predictor porque es un atributo del caso conocido desde el inicio.

### Codificación, faltantes y separación

- Las variables categóricas se transformaron con One-Hot Encoding.
- Los valores categóricos faltantes se imputaron con la categoría más frecuente y los valores numéricos faltantes con la mediana, dentro del pipeline de entrenamiento.
- No se aplicó escalamiento porque el árbol de decisión no lo requiere.
- La clase positiva es minoritaria: 390 de 2.481 casos (15,72 %). Por eso no se interpreta el accuracy por sí solo.
- Los casos se ordenaron por fecha de inicio. Se usó el 80 % más antiguo para entrenamiento (1.984 casos) y el 20 % más reciente para prueba (497 casos). Esta separación temporal simula aprender de incidentes pasados y evaluar sobre incidentes posteriores.
- Se mantuvo una sola fila por incidente, de modo que un caso no aparece en ambos conjuntos.

## 7. Modelo o modelos utilizados

### 7.1 Modelo base (baseline)

Se utilizó `DummyClassifier(strategy="most_frequent")`, que siempre predice la clase mayoritaria del conjunto de entrenamiento: que el incidente cumplirá su SLA. Este baseline permite comprobar si el árbol aporta algo más que predecir siempre el resultado más común.

### 7.2 Modelo entrenado

Se utilizó un `DecisionTreeClassifier` con `class_weight="balanced"` y `random_state=42`. Se compararon profundidades 3, 4 y 5 sobre el conjunto de validación temporal, utilizando F1 de la clase de incumplimiento para seleccionar la profundidad. Se eligió `max_depth=3` (F1 de validación 0,4944), que ofrece reglas relativamente sencillas y considera el desbalance de clases. Después se entrenó de nuevo con los primeros 1.984 casos y se evaluó una sola vez en la prueba final.

Resultados sobre el conjunto de validación (496 casos):

| Profundidad | Accuracy | Precision | Recall | F1-score |
| ---: | ---: | ---: | ---: | ---: |
| 3 | 0,7278 | 0,3646 | 0,7674 | 0,4944 |
| 4 | 0,7601 | 0,3893 | 0,6744 | 0,4936 |
| 5 | 0,7802 | 0,4109 | 0,6163 | 0,4930 |

Se eligió la profundidad 3 por tener el mayor F1 en validación (la diferencia fue pequeña). Después, el modelo se volvió a ajustar con entrenamiento + validación y se evaluó en el conjunto de prueba, que no se usó para la selección.

## 8. Evaluación

La prueba contiene 497 incidentes: 416 cumplieron el SLA y 81 lo incumplieron.

| Modelo | Accuracy | Precision | Recall | F1-score |
| --- | ---: | ---: | ---: | ---: |
| Baseline (clase mayoritaria) | 0,8370 | 0,0000 | 0,0000 | 0,0000 |
| Árbol balanceado, profundidad 3 | 0,6781 | 0,3110 | 0,8025 | 0,4483 |

El baseline obtiene 83,70 % de accuracy porque todos los casos de prueba se predicen como cumplimiento, pero no detecta ningún incumplimiento. Por lo tanto, su recall y F1 para la clase de incumplimiento son cero.

El árbol balanceado alcanza 67,81 % de accuracy, 31,10 % de precision, 80,25 % de recall y 44,83 % de F1-score. Detecta 65 de los 81 incumplimientos en el conjunto de prueba. El accuracy es más bajo que el del baseline, pero el árbol identifica muchos más incumplimientos porque se entrenó dando más peso a la clase minoritaria.

### Matriz de confusión

![Matriz de confusión del árbol de decisión](img/matriz_confusion_arbol.png)

| | Predice cumple | Predice incumple |
| --- | ---: | ---: |
| Real: cumple | 272 | 144 |
| Real: incumple | 16 | 65 |

El modelo produjo 65 verdaderos positivos, 16 falsos negativos, 144 falsos positivos y 272 verdaderos negativos. Detectó 65 de los 81 incumplimientos, pero generó bastantes falsas alarmas. Un falso negativo es un incidente que sí incumple el SLA pero que el modelo no señaló; un falso positivo genera una alerta para un incidente que finalmente sí cumple.

### Comparación de métricas

![Comparación entre baseline y árbol](img/comparacion_modelos.png)

## 9. Interpretación del modelo

El árbol balanceado detecta 65 de los 81 incumplimientos de prueba (recall de 80,25 %), mucho más que el baseline, que no detecta ninguno. A cambio, genera 144 falsas alarmas y su precision es de 31,10 %: muchas alertas corresponden a casos que finalmente cumplen el SLA. Puede servir como filtro inicial para revisión humana, pero no conviene automatizar decisiones con este desempeño sin una evaluación adicional.

Las características más importantes según el árbol fueron:

| Característica codificada | Importancia aproximada |
| --- | ---: |
| `sla_horas` | 88,53 % |
| `minutos_hasta_diagnostico` | 6,32 % |
| Grupo `Soporte_Aplicaciones` en el Diagnóstico | 3,38 % |
| Categoría `Red` | 1,78 % |

![Importancia de variables del árbol](img/importancia_variables.png)

La importancia indica cuánto contribuyó cada característica codificada a las divisiones del árbol; no demuestra causalidad. `sla_horas` destaca porque la etiqueta se define comparando el tiempo hasta la primera Resolución con ese límite. El tiempo hasta Diagnóstico también aporta información sobre cuánto tiempo del SLA ya transcurrió.

Las limitaciones incluyen el carácter sintético de los datos, la posible existencia de patrones específicos de la simulación, los valores faltantes de grupo asignado y los falsos negativos. El desempeño no debe generalizarse automáticamente a una mesa de servicio real.

## 10. Integración Process Mining + Machine Learning

1. **¿Qué hallazgo del proceso dio origen al problema de ML?** El análisis identificó variantes y una asociación entre las reasignaciones, mayores duraciones e incumplimientos. Esto motivó estudiar si el riesgo se puede anticipar desde el primer Diagnóstico.
2. **¿Qué variables del proceso ayudaron al modelo?** Se incluyeron como candidatas la duración y la cantidad de eventos observados hasta el Diagnóstico, las reasignaciones y escalamientos ya ocurridos, y el grupo o recurso registrado en ese momento. En el árbol final, las características con importancia no nula fueron `sla_horas` (88,53 %), `minutos_hasta_diagnostico` (6,32 %), el grupo `Soporte_Aplicaciones` en el Diagnóstico (3,38 %) y la categoría `Red` (1,78 %).
3. **¿El modelo confirma o contradice Process Mining?** Es compatible con la idea de que el tiempo consumido antes de la resolución y las características del caso aportan información. Sin embargo, el desempeño moderado y la importancia de las variables no demuestran que las reasignaciones causen incumplimientos.
4. **¿Qué aporta ML que no era visible solo en el mapa?** Produce una predicción cuantificable para casos individuales usando atributos disponibles en un momento concreto y permite medir los falsos positivos y falsos negativos.
5. **¿Qué aporta Process Mining que no se ve solo mediante ML?** Expone el orden real de las actividades, las rutas alternativas, las repeticiones, las reasignaciones y las transiciones donde se acumula tiempo. Esto da contexto operativo a las predicciones.

## 11. Conclusiones

1. El conjunto original tiene 21.022 eventos. Tras quitar duplicados exactos, excluir timestamps inválidos del análisis temporal y estandarizar actividades, se obtuvieron 20.899 eventos válidos y 12 actividades estandarizadas.
2. La variable de incumplimiento pudo calcularse para 2.488 casos con registro y primera Resolución válidos: 390 incumplieron el SLA (15,68 %). Para predecir después del primer Diagnóstico, la cohorte se redujo a 2.481 casos, manteniendo los 390 incumplimientos.
3. El árbol balanceado tuvo menor accuracy que el baseline (67,81 % frente a 83,70 %), pero detectó 65 de 81 incumplimientos en prueba. Generó 144 falsas alarmas y omitió 16 incumplimientos; por ello, sus predicciones solo deben servir como apoyo y requieren revisión humana.
4. El límite SLA y el tiempo transcurrido hasta el primer Diagnóstico fueron las características más importantes del árbol. Esto respalda la utilidad de considerar tanto el compromiso de servicio como el tiempo ya consumido.
5. Las diferencias observadas entre casos con y sin reasignaciones son asociaciones en datos sintéticos; se requieren análisis adicionales antes de atribuirles causalidad.

## 12. Recomendaciones

### Recomendación 1: crear una alerta de riesgo después del primer Diagnóstico

- **Qué modificar:** incorporar una revisión o alerta de riesgo para los incidentes que, según el modelo, podrían incumplir el SLA.
- **Parte del proceso:** inmediatamente después del primer `Diagnóstico`.
- **Por qué:** el modelo alcanza 80,25 % de recall y detecta 65 de los 81 incumplimientos del conjunto de prueba.
- **Evidencia y precaución:** el modelo puede apoyar la priorización, pero omite 16 incumplimientos y genera 144 falsas alarmas. Debe probarse en paralelo con la revisión humana antes de automatizar decisiones.

### Recomendación 2: revisar los incidentes que requieren reasignación

- **Qué modificar:** evaluar las reglas de enrutamiento inicial por categoría y grupo para reducir transferencias innecesarias.
- **Parte del proceso:** asignación inicial y transiciones de Reasignación a Diagnóstico.
- **Por qué:** los casos con reasignaciones tuvieron duración media de 21,92 horas y tasa de incumplimiento de 31,70 %, frente a 11,46 horas y 10,23 % en casos sin reasignación.
- **Evidencia y precaución:** la asociación justifica revisar el enrutamiento, pero no prueba que quitar reasignaciones por sí solo reduzca los incumplimientos. Debe evaluarse el efecto después de cualquier cambio.

## 13. Resumen ejecutivo

Este trabajo analiza el proceso sintético de una mesa de servicio de TI, cuyo objetivo es atender incidentes dentro de los límites de tiempo comprometidos por el SLA. En el Corte 2 se identificaron 51 variantes del proceso y se observó que los incidentes con reasignaciones tendían a presentar mayores duraciones y tasas de incumplimiento. La ruta más frecuente fue la secuencia directa desde el registro hasta el cierre, con 1.378 casos (55,12 %).

Para el Corte 3 se formuló un problema de clasificación: predecir si un incidente incumplirá el SLA utilizando la información disponible inmediatamente después del primer Diagnóstico. La etiqueta se construyó comparando las horas entre el primer registro y la primera Resolución con `sla_horas`. De los 2.488 casos con etiqueta calculable, se incluyeron 2.481 que tenían un primer Diagnóstico válido anterior o igual a la primera Resolución; 390 de estos incumplieron el SLA.

Se comparó un baseline que siempre predice la clase mayoritaria con un árbol de decisión balanceado. La profundidad 3 se seleccionó con un conjunto temporal de validación, comparando F1 para profundidades 3, 4 y 5. En una prueba temporal de 497 incidentes, el baseline logró 83,70 % de accuracy, pero detectó cero incumplimientos. El árbol balanceado alcanzó 67,81 % de accuracy, 31,10 % de precision, 80,25 % de recall y 44,83 % de F1-score, detectando 65 de 81 incumplimientos y omitiendo 16, aunque generó 144 falsas alarmas.

Las características más importantes fueron `sla_horas` y el tiempo transcurrido hasta Diagnóstico. Se recomienda evaluar una alerta asistida después del Diagnóstico y revisar los criterios de reasignación. Debido a los falsos negativos, las falsas alarmas y el carácter sintético de los datos, el modelo debe utilizarse como apoyo a la toma de decisiones y no como sustituto del personal de soporte.

---

## 14. Declaración de uso de IA generativa

| Herramienta | Para qué se utilizó | En qué parte del trabajo |
| --- | --- | --- |
| ChatGPT | Apoyo para revisar el código, comprobar coherencia de los resultados, estructurar el informe y redactar explicaciones | Preparación, evaluación, interpretación y redacción de las secciones del Corte 3 |
| Gemini | Apoyo para generar y ejecutar el notebook de análisis en Google Colab | Construcción y ejecución del notebook de Machine Learning |

Ambos integrantes deben poder explicar cualquier parte del trabajo, incluida la generada con apoyo de IA.
