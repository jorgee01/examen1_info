# Corte 2 -- Process Mining

## 1. Integrantes

-   Integrante 1: Jorge Luis Murillo Valverde, C35519
## 2. Caso seleccionado

**Caso:** Caso 04 -- Mesa de servicio de TI.

Archivo de origen: `datos/raw/caso_04_soporte_ti.csv`.

## 3. Descripción de los datos utilizados

-   **Cantidad de registros originales:** 21.022 eventos.
-   **Cantidad de registros del Event Log preparado:** 20.899 eventos.
-   **Cantidad de casos:** 2.500 incidentes.
-   **Cantidad de actividades:** 12 actividades estandarizadas a partir
    de 38 formas de escritura originales.
-   **Rango temporal válido:** del 02/02/2026 01:05:28 al 29/06/2026
    23:59:47.
-   **Variables principales:** `case_id`, `activity`, `timestamp`,
    `resource`, `grupo_asignado`, `prioridad`, `categoria`, `canal`,
    `departamento` y `sla_horas`.
-   **Problemas de calidad detectados:**
    -   Se identificaron 86 registros duplicados exactos.
    -   Se encontraron 37 eventos con `timestamp` inválido o vacío.
    -   Las actividades presentaban diferencias de mayúsculas, tildes y
        espacios; se estandarizaron 38 formas originales en 12 nombres
        de actividad.
    -   `grupo_asignado` está vacío en 11.117 eventos del Event Log
        preparado. No se imputaron automáticamente esos valores.
    -   No se encontraron inconsistencias en los valores de `prioridad`,
        `categoria`, `canal`, `departamento` ni `sla_horas` dentro de un
        mismo caso.

Los datos son sintéticos y tienen fecha de corte el 30/06/2026. Algunos
incidentes pueden estar incompletos al finalizar el período. Las
métricas de duración y SLA se calcularon para los casos con marcas
temporales válidas para el inicio y la primera resolución.

## 4. Preparación de datos

  --------------------------------------------------------------------------------
  Transformación          Motivo             Efecto sobre los datos
  ----------------------- ------------------ -------------------------------------
  Estandarización de      Evitar que la      Se unificaron 38 formas originales en
  espacios, mayúsculas y  misma actividad se 12 actividades
  tildes en las           cuente como        
  actividades             actividades        
                          diferentes         

  Eliminación de          Evitar contar dos  Se eliminaron 86 filas duplicadas
  registros duplicados    veces el mismo     
  exactos                 evento             

  Conversión de           Validar marcas     Se identificaron 37 timestamps
  `timestamp` a fecha y   temporales y       inválidos
  hora                    calcular           
                          intervalos         

  Exclusión de eventos    No es posible      Se excluyeron 37 eventos; los 2.500
  sin timestamp válido    ordenarlos         casos conservan otros eventos válidos
  del análisis de         cronológicamente   
  secuencias y tiempos    de forma confiable 

  Ordenamiento por        Reconstruir la     El Event Log quedó ordenado por caso
  `case_id` y `timestamp` secuencia temporal y tiempo
                          de cada incidente  

  Revisión de atributos a Detectar valores   No se encontraron inconsistencias en
  nivel de caso           inconsistentes     `prioridad`, `categoria`, `canal`,
                          dentro de un mismo `departamento` ni `sla_horas`
                          incidente          
  --------------------------------------------------------------------------------

Los timestamps inválidos no se corrigieron con fechas inventadas. Se
excluyeron de las métricas y secuencias temporales para evitar alterar
artificialmente el orden del proceso.

## 5. Event Log final

-   **Notebook o script de `scripts/` donde se construye el Event Log:**
    `scripts/01_corte2_process_mining.ipynb`.
-   **Archivo de `datos/processed/` donde se guardó:**
    `datos/processed/caso_04_soporte_ti_event_log.csv`.

Indique claramente:

-   **Case ID:** `case_id`.
-   **Activity:** `activity`.
-   **Timestamp:** `timestamp`.
-   **Otros atributos utilizados:** `resource`, `grupo_asignado`,
    `prioridad`, `categoria`, `canal`, `departamento` y `sla_horas`.

### Muestra representativa del Event Log

La siguiente muestra corresponde al incidente `INC00001`.

  ------------------------------------------------------------------------------------------
  case_id     activity        timestamp    resource       grupo_asignado         prioridad
  ----------- --------------- ------------ -------------- ---------------------- -----------
  INC00001    Incidente       2026-06-01   Sistema        ---                    P4
              registrado      09:12:02                                           

  INC00001    Clasificación   2026-06-01   N1_Agente_01   ---                    P4
                              11:56:21                                           

  INC00001    Asignación      2026-06-01   Aplic_Tec01    Soporte_Aplicaciones   P4
                              12:27:35                                           

  INC00001    Diagnóstico     2026-06-01   Aplic_Tec04    Soporte_Aplicaciones   P4
                              19:13:00                                           

  INC00001    Resolución      2026-06-02   Aplic_Tec04    Soporte_Aplicaciones   P4
                              04:45:04                                           

  INC00001    Validación      2026-06-02   Usuario        ---                    P4
                              09:25:42                                           

  INC00001    Cierre          2026-06-03   Sistema        ---                    P4
                              09:25:43                                           
  ------------------------------------------------------------------------------------------

El guion representa un valor vacío de `grupo_asignado` en el archivo
original.

## 6. Descubrimiento del proceso

**Modelo o mapa de proceso generado:**
`entregas/corte_2_process_mining/img/mapa_proceso_dfg.png`.

**Método utilizado:** grafo Directly-Follows (DFG) construido a partir
de las transiciones observadas en el Event Log preparado con PM4Py. Para
facilitar la lectura, el mapa muestra transiciones con una frecuencia de
al menos 50 eventos.

![Grafo Directly-Follows del proceso de soporte de
TI](img/mapa_proceso_dfg.png)

### Interpretación

La ruta principal observada es:

**Incidente registrado → Clasificación → Asignación → Diagnóstico →
Resolución → Validación → Cierre.**

También aparecen rutas alternativas. Desde Diagnóstico puede haber
Reasignación o Escalamiento y luego un retorno a Diagnóstico. Algunos
incidentes quedan Pendientes de usuario antes de registrar una Respuesta
de usuario; otros se reabren después de la validación y regresan al
diagnóstico.

Las transiciones más frecuentes son Resolución → Validación (2.671
eventos), Validación → Cierre (2.496), Clasificación → Asignación
(2.492), Incidente registrado → Clasificación (2.491) y Asignación →
Diagnóstico (2.490). El mapa omite transiciones con frecuencia inferior
a 50, por lo que no representa todas las rutas excepcionales.

## 7. Variantes

-   **Cantidad de variantes:** 51.
-   **Variante más frecuente:** Incidente registrado → Clasificación →
    Asignación → Diagnóstico → Resolución → Validación → Cierre.
-   **Frecuencia de la variante más frecuente:** 1.378 casos.
-   **Porcentaje de la variante más frecuente:** 55,12 % de los casos.
-   **Principales diferencias entre variantes:** reasignaciones, espera
    de respuesta del usuario, escalamientos, reaperturas y repetición de
    actividades de diagnóstico o resolución.

  -------------------------------------------------------------------------
            Posición Variante                      Casos         Porcentaje
  ------------------ ---------------- ------------------ ------------------
                   1 Ruta principal                1.378            55,12 %
                     sin                                 
                     reasignaciones                      
                     ni reaperturas                      

                   2 Ruta principal                  359            14,36 %
                     con una                             
                     Reasignación y                      
                     retorno a                           
                     Diagnóstico                         

                   3 Diagnóstico →                   255            10,20 %
                     Pendiente de                        
                     usuario →                           
                     Respuesta de                        
                     usuario antes de                    
                     Resolución                          

                   4 Diagnóstico →                    80             3,20 %
                     Escalamiento →                      
                     Diagnóstico                         
                     antes de                            
                     Resolución                          

                   5 Resolución →                     79             3,16 %
                     Validación →                        
                     Reapertura →                        
                     Diagnóstico →                       
                     Resolución →                        
                     Validación →                        
                     Cierre                              
  -------------------------------------------------------------------------

Las cinco variantes más frecuentes reúnen 2.151 casos (86,04 %). Estos
resultados describen el Event Log preparado; las secuencias pueden verse
afectadas por eventos excluidos debido a timestamps inválidos.

## 8. Análisis temporal

La duración hasta la primera resolución se calculó desde
`Incidente registrado` hasta la primera actividad `Resolución`, según la
definición de SLA del caso.

  -----------------------------------------------------------------------
  Métrica                                                       Resultado
  ------------------------------ ----------------------------------------
  Casos con inicio y primera                                        2.488
  resolución válidos             

  Duración promedio hasta la                                  14,11 horas
  primera resolución             

  Duración mínima                                              1,28 horas

  Duración máxima                                             93,61 horas

  Mediana hasta la primera                                    10,81 horas
  resolución                     

  Duración promedio desde el                    46,38 horas (2.493 casos)
  registro hasta el último       
  cierre válido                  
  -----------------------------------------------------------------------

### Actividades con mayor espera y transiciones problemáticas

Los intervalos representan el tiempo transcurrido entre marcas de tiempo
consecutivas. No distinguen por sí solos el tiempo de espera del tiempo
de trabajo activo.

  -----------------------------------------------------------------------
  Transición     Eventos observados    Espera promedio            Mediana
  -------------- ------------------ ------------------ ------------------
  Validación →                2.496        24,00 horas        24,00 horas
  Cierre                                               

  Pendiente de                  384        14,66 horas        11,64 horas
  usuario →                                            
  Respuesta de                                         
  usuario                                              

  Resolución →                2.671         7,18 horas         6,00 horas
  Validación                                           

  Reasignación →                814         5,53 horas         4,16 horas
  Diagnóstico                                          

  Asignación →                2.490         4,27 horas         3,11 horas
  Diagnóstico                                          

  Diagnóstico →               2.284         2,90 horas         2,05 horas
  Resolución                                           
  -----------------------------------------------------------------------

### Posibles cuellos de botella

-   **Validación → Cierre:** presenta un intervalo promedio de 24 horas
    en 2.496 transiciones. Conviene verificar si corresponde a una regla
    del proceso simulado antes de interpretarlo como demora evitable.
-   **Pendiente de usuario → Respuesta de usuario:** promedia 14,66
    horas; la espera por respuesta del usuario es un punto relevante
    para estudiar.
-   **Resolución → Validación:** el promedio es de 7,18 horas, superior
    al intervalo entre Diagnóstico y Resolución (2,90 horas).
-   **Reasignación → Diagnóstico:** presenta una espera promedio de 5,53
    horas. Los casos con reasignaciones también muestran mayor duración
    promedio hasta la primera resolución.

## 9. Verificación de cumplimiento (conformance)

**Referencia utilizada y motivo:** la ruta principal descrita para el
Caso 04 en el Corte 1: Incidente registrado → Clasificación → Asignación
→ Diagnóstico → Resolución → Validación → Cierre. Se consideran
Reasignación y Escalamiento como pasos alternativos observados en los
datos.

**Técnica aplicada:** verificación de reglas con Python sobre las
secuencias ordenadas por caso. Se revisó la presencia de actividades
principales, el orden de sus primeras apariciones y las actividades
adicionales. No se aplicó token-based replay ni alineamientos.

**Desviaciones encontradas:**

  ---------------------------------------------------------------------------
  Desviación o revisión               Cantidad de casos Interpretación
  ------------------------ ---------------------------- ---------------------
  Sin evento válido de                                4 El evento pudo
  `Incidente registrado`                                excluirse por
                                                        timestamp inválido;
                                                        no demuestra que el
                                                        proceso comenzara sin
                                                        registro

  Sin `Clasificación`                                 5 Ausencia en el Event
  válida                                                Log preparado;
                                                        revisar junto con los
                                                        timestamps inválidos

  Sin `Asignación` válida                             3 Ausencia en el Event
                                                        Log preparado

  Sin `Diagnóstico` válido                            6 Puede afectar la
                                                        evaluación del orden

  Sin `Resolución` válida                             8 No se puede comprobar
                                                        la ruta completa ni
                                                        calcular el SLA para
                                                        esos casos

  Sin `Validación` válida                             0 Todos los casos
                                                        tienen al menos una
                                                        Validación observada

  Sin `Cierre` válido                                 3 Pueden estar
                                                        incompletos o tener
                                                        un evento de cierre
                                                        inválido

  Inversión aparente en el               1 (`INC01170`) Un evento Diagnóstico
  orden de la ruta                                      tiene timestamp
  principal                                             inválido; no se puede
                                                        concluir que sea una
                                                        desviación real

  Actividades adicionales    Pendiente de usuario: 385; Rutas adicionales
                             Respuesta de usuario: 384; observadas, no
                                        Reapertura: 181 errores automáticos
  ---------------------------------------------------------------------------

**Respuesta:**

Las ausencias se refieren al Event Log preparado después de excluir
eventos sin timestamp válido. Por ello, no se consideran automáticamente
incumplimientos reales del procedimiento: algunos eventos pudieron
existir en el archivo original, pero no fue posible ubicarlos
cronológicamente. En los casos evaluables, el incumplimiento del SLA
aparece con mayor frecuencia en incidentes de prioridad alta y en los
que tuvieron reasignaciones. También hay diferencias por canal y
categoría; estas asociaciones no demuestran causalidad.

## 10. Hallazgos

### Hallazgo 1

**Evidencia:** la variante principal, sin reasignaciones ni reaperturas,
aparece en 1.378 casos (55,12 %). Las cinco variantes más frecuentes
representan el 86,04 % de los casos.

**Interpretación:** más de la mitad de los incidentes siguen una ruta
directa, pero una parte importante requiere pasos adicionales. Las
reasignaciones, esperas de usuario, escalamientos y reaperturas son
diferencias útiles para segmentar el análisis.

### Hallazgo 2

**Evidencia:** de los 2.488 incidentes con inicio y primera resolución
válidos, 390 incumplieron el SLA (15,68 %).

  -----------------------------------------------------------------------
  Prioridad        Casos evaluables    Incumplimientos            Tasa de
                                                           incumplimiento
  -------------- ------------------ ------------------ ------------------
  P1                            104                 53            50,96 %

  P2                            511                219            42,86 %

  P3                          1.079                114            10,57 %

  P4                            794                  4             0,50 %
  -----------------------------------------------------------------------

**Interpretación:** el incumplimiento es más frecuente en las
prioridades P1 y P2, que tienen límites de SLA más cortos. La prioridad
es una variable relevante para describir el riesgo, aunque las tasas
también dependen de la distribución y las características de los
incidentes.

### Hallazgo 3

**Evidencia:** entre los casos evaluables, los que no tuvieron
reasignaciones (1.857 casos) presentan una duración promedio de 11,46
horas y una tasa de incumplimiento de 10,23 % (190 casos). Los que
tuvieron al menos una reasignación (631 casos) presentan una duración
promedio de 21,92 horas y una tasa de incumplimiento de 31,70 % (200
casos).

**Interpretación:** las reasignaciones están asociadas con una mayor
duración y una tasa de incumplimiento aproximadamente tres veces más
alta. Esto no demuestra causalidad: los incidentes más complejos también
pueden necesitar más transferencias.

## 11. Implicaciones para Machine Learning

-   **Resultado útil para predecir o explicar:** la probabilidad de
    incumplimiento del SLA, comparando el tiempo entre
    `Incidente registrado` y la primera `Resolución` con `sla_horas`. La
    etiqueta se pudo evaluar en 2.488 casos con inicio y primera
    resolución válidos; los demás no deben etiquetarse automáticamente
    como cumplidos o incumplidos.
-   **Variables que podrían utilizarse:** `prioridad`, `categoria`,
    `canal`, `departamento`, `sla_horas` y el grupo asignado
    inicialmente, siempre que estén disponibles al momento de la
    predicción.
-   **Variables derivadas del proceso que podrían construirse:**
    duración acumulada hasta el momento de predicción, cantidad de
    actividades observadas, número de repeticiones, cantidad de
    reasignaciones y escalamientos observados, presencia de determinadas
    actividades, variante parcial del proceso y tiempo desde el último
    evento.

Para evitar fuga de información (*data leakage*), solo se deben utilizar
variables disponibles al momento de predecir. El tiempo final hasta la
resolución, el resultado final del SLA y el número total de
reasignaciones no deben usarse para una alerta temprana si todavía no se
conocen.

## 12. Conclusión del corte

El análisis del Caso 04 permitió reconstruir una ruta principal de
atención e identificar 51 variantes en el Event Log preparado. La ruta
más frecuente representa el 55,12 % de los casos; otras variantes
incluyen reasignaciones, escalamientos, esperas de usuario y
reaperturas. La duración promedio hasta la primera resolución fue de
14,11 horas en los casos evaluables, y el 15,68 % incumplió el SLA.

Los incidentes con reasignaciones mostraron mayores duraciones y más
incumplimientos. Las prioridades P1 y P2 también presentaron tasas de
incumplimiento más altas. Se observaron intervalos relevantes entre
Validación y Cierre, y entre la espera y la respuesta del usuario. Estos
resultados son asociaciones sobre datos sintéticos y no demuestran
causalidad en una organización real.

La preparación mostró problemas de calidad, incluidos duplicados,
timestamps inválidos y diferencias de escritura en los nombres de
actividades. Los eventos con tiempos inválidos se excluyeron de las
secuencias temporales. Para el siguiente corte, se propone evaluar si
los atributos conocidos al inicio y las características del proceso
observadas hasta un momento definido permiten predecir el incumplimiento
del SLA, evitando la fuga de información.

------------------------------------------------------------------------

## 13. Declaración de uso de IA generativa

  -----------------------------------------------------------------------
  Herramienta    Para qué se utilizó      En qué parte del trabajo
  -------------- ------------------------ -------------------------------
  ChatGPT        Apoyo para analizar el   Preparación de datos y
                 archivo de datos,        redacción de las secciones 3 a
                 calcular estadísticas    12
                 descriptivas y preparar  
                 el borrador del          
                 documento                

  -----------------------------------------------------------------------

Ambos integrantes deben poder explicar cualquier parte del trabajo,
incluida la generada con apoyo de IA.
