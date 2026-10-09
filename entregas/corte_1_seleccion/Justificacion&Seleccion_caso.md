# Corte 1 – Selección y justificación del caso

## 1. Integrantes

- Integrante: Jorge Luis Murillo Valverde
- Carnét: C35519

## 2. Revisión de los casos

| Caso              | Problema principal identificado                                                                | ¿Permite Process Mining? ¿Por qué?                                                                                                 | Posible uso posterior de ML                                                                            | Nivel de interés : Alto/Medio/Bajo |
| ----------------- | ---------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| 01. Salud         | Posibles demoras en la atención de pacientes.                                                  | Sí, porque cada paciente recorre varias etapas con fecha y hora.                                                                   | Predecir qué pacientes tendrán una atención más larga o un resultado distinto.                         | Bajo                               |
| 02. Pedidos       | Posibles atrasos en el despacho y la entrega.                                                  | En efecto, porque cada pedido pasa por etapas ordenadas en el tiempo.                                                              | Predecir qué pedidos llegarán tarde.                                                                   | Bajo                               |
| 03. Créditos      | Posibles demoras en la evaluación y aprobación de solicitudes.                                 | Así es, porque cada solicitud sigue una secuencia de revisiones.                                                                   | Predecir el resultado o la demora de una solicitud.                                                    | Medio                              |
| 04. Soporte TI    | Incidentes que pasan por varias reasignaciones o escalamientos y que podrían incumplir el SLA. | Sí. El archivo es un registro de eventos con identificador de incidente, actividad, fecha y hora, y grupo asignado en cada evento. | Predecir si un incidente incumplirá el SLA, usando solo información disponible al momento de predecir. | Alto                               |
| 05. Académico     | Posible avance o abandono de estudiantes dentro de un curso.                                   | Probablemente sí, si cada estudiante se toma como un caso con eventos en el tiempo.                                                | Predecir el resultado final del estudiante.                                                            | Bajo                               |
| 06. Mantenimiento | Posibles reparaciones largas o fallas que se repiten.                                          | Seguramente. Si cada orden de trabajo se toma como un caso.                                                                        | Estimar el tiempo de reparación o anticipar fallas repetidas.                                          | Bajo                               |

## 3. Caso seleccionado

**Número y nombre del caso:** Caso 04 – Mesa de servicio de TI

**Archivo de datos:** `datos/raw/caso_04_soporte_ti.csv` (aproximadamente 2 500 incidentes, de febrero a junio de 2026, con fecha de corte el 30/06/2026)

## 4. Justificación de la selección

Se eligió el Caso 04, Mesa de servicio de TI, porque retrata una situación que casi todas las personas han vivido: reportar una falla y quedarse esperando mientras el problema pasa de mano en mano sin que nadie lo resuelva. Además, los datos permiten reconstruir paso a paso esa historia y ver dónde se atora. Una mesa de servicio es el equipo al que se acude cuando algo de tecnología falla, por ejemplo “no puedo entrar a mi correo”. Cada problema que se reporta se llama incidente. Cuando un incidente pasa de un grupo de soporte a otro, tarda más en resolverse y puede pasarse del tiempo límite que tiene para arreglarse, llamado SLA (acuerdo de nivel de servicio). Es como si a una persona le prometen arreglar su internet en 2 horas y lo arreglan en 10: no se cumplió lo prometido.

Cada fila del archivo de datos es un evento, es decir, algo que le pasó a un incidente. Los pasos típicos son: registro, clasificación, asignación (se le da a un grupo de soporte), diagnóstico (se busca la causa), reasignación (se pasa a otro grupo), escalamiento (se sube a alguien con más experiencia), resolución (se arregla), validación (se confirma que quedó bien) y cierre (se termina el caso). Los nombres exactos de estos pasos todavía deben comprobarse en el archivo.

Los datos tienen lo necesario para armar la historia de cada incidente. Todos los archivos incluyen cuatro columnas: case_id, que es el número que identifica cada incidente; activity, que es el paso que ocurrió; timestamp, que es la fecha y hora en que ocurrió; y resource, que es la persona o recurso que lo hizo. Con el case_id y el timestamp se pueden ordenar los pasos de cada incidente y calcular cuánto tiempo pasa entre uno y otro. Como el archivo viene ordenado por fecha y no por incidente (como tener las fotos de un álbum mezcladas), primero hay que ordenarlo por case_id y timestamp antes de analizarlo.

Este caso además trae columnas extra: grupo_asignado, que es el grupo de soporte responsable en cada momento y sirve para ver cuándo el incidente pasa de un grupo a otro; prioridad, que indica qué tan urgente es, de P1 (crítica) a P4 (baja); categoria, que es el tipo de problema; canal, que es cómo se reportó; departamento, que es quién lo reportó; y sla_horas, que son las horas que se tienen para resolverlo. Estas columnas sirven para comparar los recorridos y los tiempos según el tipo de incidente.

Con estos elementos se esperan encontrar recorridos distintos: incidentes que se resuelven directamente y otros que rebotan por varios grupos antes de resolverse. Esto debe confirmarse al revisar el archivo.

También se puede hacer un análisis con Machine Learning, que consiste en enseñarle a una computadora a encontrar patrones. El tiempo se mide en horas corridas desde que se registra el incidente hasta que se resuelve por primera vez, y se compara con el sla_horas de cada incidente. Así se puede calcular si cada uno cumplió o no el SLA (sí o no), y usar ese resultado para que la computadora aprenda a predecir qué incidentes tienen más riesgo de incumplir.

Hay que tener en cuenta algunas limitaciones. Primero, los datos son sintéticos, es decir, inventados por una simulación y no provienen de una empresa real, así que lo que se encuentre solo habla de este archivo y no de una organización concreta. Segundo, los datos no están limpios, porque pueden tener errores o cosas raras. Tercero, los datos se extrajeron con fecha de corte el 30/06/2026, así que algunos incidentes pueden no haber terminado todavía. Por último, la cantidad de reasignaciones, la cantidad de escalamientos y el cumplimiento del SLA no vienen como columnas, así que deben calcularse a partir de los eventos y de sla_horas.

---

## 5. Comprensión inicial del proceso

### 5.1 ¿Cuál considera que es el inicio del proceso?
El inicio es la actividad "Incidente registrado". Es el punto desde el cual se mide el SLA, por lo que se espera que sea el primer evento de cada incidente.
### 5.2 ¿Cuál considera que es el final del proceso?
Espero que el proceso termine con el cierre del incidente, después de la resolución y, cuando corresponda, la validación. Hay que tener en cuenta que el SLA se mide solo hasta la primera "Resolución", no hasta el cierre. Además, algunos incidentes pueden no tener cierre porque seguían abiertos en la fecha de corte.
### 5.3 ¿Qué actividades principales espera encontrar?
Según la descripción del caso, espero encontrar incidente registrado, clasificación, asignación, diagnóstico, reasignación, escalamiento, resolución, validación y cierre. Todavía debo comprobar en el archivo cómo se llaman realmente y si alguna se repite dentro de un mismo incidente, por ejemplo varias reasignaciones.

### 5.4 Proponga los componentes iniciales del Event Log

| Componente | Variable o propuesta | Justificación |
|---|---|---|
| Case ID | `case_id` | Identifica cada incidente y agrupa todos sus eventos. Está indicada en el README de los datos. |
| Activity | `activity` | Indica qué ocurrió en cada evento y permite construir las rutas del proceso. Los valores concretos deben revisarse en el archivo. |
| Timestamp | `timestamp` | Permite ordenar los eventos y calcular el tiempo entre actividades y hasta la primera resolución. |
| Recurso, si aplica | `resource` y `grupo_asignado` | `resource` indica quién o qué ejecutó el evento y `grupo_asignado` el grupo responsable en ese momento, lo que permite detectar traspasos entre grupos. Falta revisar qué contiene `resource` en este caso. |
| Otros atributos | `prioridad`, `categoria`, `canal`, `departamento`, `sla_horas` | Permiten comparar recorridos y tiempos según el tipo de incidente y calcular si se cumplió el SLA. El número de reasignaciones y escalamientos no viene en el archivo y se calculará a partir de los eventos. |

---

## 6. Preguntas iniciales para Process Mining
### Pregunta 1
¿Hay algún departamento o canal de reporte cuyos incidentes den más vueltas entre grupos que los demás?
### Pregunta 2
Hay incidentes que se pasan del tiempo permitido sin haber cambiado nunca de grupo?
### Pregunta 3
¿Dónde se pierde más tiempo: antes de asignar el incidente o después de pasarlo a otro grupo?

---

## 7. Hipótesis o sospechas iniciales
Ejemplos de elementos a considerar:

- retrasos;
- reprocesos;
- actividades repetidas;
- variantes;
- cuellos de botella;
- incumplimientos;
- rutas excepcionales.

Es muy probable que los incidentes con una o más reasignaciones tardan más en llegar a la primera resolución que los que no tienen ninguna, y que esto se refleja en un mayor porcentaje de incumplimiento del SLA. También sospecho que las mayores demoras se concentran en los intervalos posteriores a una asignación o reasignación, y no en el tramo final hacia la resolución, aunque con una sola marca de tiempo por evento no podré separar con certeza el tiempo de espera del tiempo de trabajo. Por último, creo que los incidentes de prioridad P1 siguen recorridos distintos a los de prioridad P4, y que pueden existir rutas excepcionales con muchas reasignaciones o actividades repetidas. Todas estas ideas son sospechas y deberán comprobarse con los datos.

---

## 8. Posible problema futuro de Machine Learning

### Posible variable objetivo

Incumplimiento del SLA (sí o no). No viene como columna, por lo que se calcularía comparando el tiempo entre "Incidente registrado" y la primera "Resolución" con `sla_horas`. Queda pendiente decidir cómo tratar los incidentes que no tenían resolución en la fecha de corte.

### Posibles variables predictoras

Las variables conocidas desde el registro del incidente: `prioridad`, `categoria`, `canal`, `departamento` y `sla_horas`. También podrían usarse datos de las primeras etapas, como el grupo asignado inicialmente o el número de reasignaciones hasta el momento de predecir. No deben usarse variables que solo se conocen después, como el tiempo total hasta la resolución o el número final de reasignaciones, porque provocarían fuga de información (*data leakage*).

### Tipo de problema que podría resultar

- [x] Clasificación
- [ ] Regresión
- [ ] Agrupamiento
- [ ] Aún no se puede determinar

### Justificación del problema anterior...

Si la variable objetivo tiene dos categorías (cumple o incumple), el problema sería de clasificación. Se elige esta opción de forma TENTATIVA, porque depende de que los datos permitan construir la etiqueta de manera confiable. Predecir qué incidentes tienen más riesgo de incumplir el SLA permitiría priorizar su atención antes de que venza el plazo. El momento de la predicción debe elegirse con cuidado: un tiempo fijo, por ejemplo dos horas, podría no tener sentido para prioridades con SLA muy corto o muy largo, por lo que quizá convenga definirlo por etapa del proceso.

---

## 9. Decisión final

El Caso 04 es una buena oportunidad para integrar ambas técnicas porque cuenta con un registro de eventos con identificador de incidente, actividades, fecha y hora, grupo responsable y un tiempo máximo de SLA para cada incidente. Con Process Mining estudiaré primero el flujo real de atención: las variantes, los puntos donde se acumulan demoras y el efecto de las reasignaciones y escalamientos. Con esos resultados podré revisar después si algunos atributos conocidos de forma temprana permiten anticipar qué incidentes tienen riesgo de incumplir el SLA, siempre que los datos lo permitan y se evite la fuga de información.

---

## 10. Declaración de uso de IA generativa

Indique si utilizaron herramientas de IA generativa (ChatGPT, Claude, Copilot, Gemini u otras) en este corte.

| Herramienta        | Para qué se utilizó                                                                                                                                                                                                                                                   | En qué parte del trabajo                    |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------- |
| ChatGPT            | Organizar la estructura del documento, mejorar la redacción y proponer estructura inicial de hipótesis.                                                                                                                                                               | Borrador inicial: secciones 2, 4, 6, 7 y 8. |
| Claude (Anthropic) | Revisar el borrador contra el `datos/README.md`, corregir campos que no estaban respaldados por los datos, ajustar el documento a la plantilla, redactar la justificación y el resto de las respuestas según mis indicaciones, y elaborar el glosario complementario. | Todo el documento en su versión final.      |

Mi aporte fue el siguiente:

- Hice el análisis y la coordinación del trabajo con las herramientas de IA: lo que hice fue organizar mejor mis ideas para estructurar mejor esta primera parte del trabajo para que quede bien planteada.
- Redacté toda la justificación y después la formalicé para que las ideas tomaran un poco más de orden. 
- Realicé un glosario complementario para aclararme sobre ciertos términos que no estaban en el glosario del repo y así poder entender mejor la estructura del proyecto-exámen.

> Ambos integrantes deben poder explicar cualquier parte del trabajo, incluida la generada con apoyo de IA.

---

## Anexo – Glosario complementario

Este glosario incluye solo los términos usados en el documento que no aparecen en el glosario oficial de recursos. Los demás términos (por ejemplo Event Log, Caso, Actividad, Recurso, Variante, Cuello de botella, SLA, Variable objetivo, Variables predictoras, Clasificación, Regresión y Fuga de información) se usan con la definición de ese glosario.

| Término | Significado |
|---|---|
| Process Mining (minería de procesos) | Técnica que reconstruye cómo se ejecuta realmente un proceso a partir de los registros que deja en un sistema. |
| Event Log sintético | Registro de eventos generado por simulación, no tomado de una organización real. |
| Timestamp | Fecha y hora en que ocurrió un evento. |
| Incidente | Interrupción o falla de un servicio de TI que un usuario reporta. |
| Mesa de servicio | Equipo que recibe y atiende las solicitudes e incidentes de los usuarios de TI. |
| Reasignación | Cambio del incidente de un técnico o grupo de soporte a otro. |
| Escalamiento | Paso del incidente a un nivel de soporte más especializado. |
| `grupo_asignado` | Columna del archivo que indica el grupo de soporte responsable del incidente en cada evento. |
| `sla_horas` | Columna con el tiempo máximo del SLA de cada incidente, según su prioridad. |
| Incumplimiento del SLA | Caso en que el tiempo real desde "Incidente registrado" hasta la primera "Resolución" supera `sla_horas`. |
| Prioridad P1–P4 | Nivel de urgencia del incidente: P1 es crítica y P4 es baja. |
| Fecha de corte | Fecha de la extracción de los datos (30/06/2026). Los incidentes abiertos en esa fecha tienen el proceso incompleto. |
| Datos no limpios | Datos con posibles errores o inconsistencias, propios de una extracción real, que deben revisarse antes de analizar. |
| CSV | Archivo de texto donde los datos se separan por comas. |
| Machine Learning (ML, aprendizaje automático) | Técnicas en las que un modelo aprende patrones a partir de datos para hacer predicciones. |
| IA generativa | Herramientas como ChatGPT o Claude que producen texto a partir de instrucciones. |
