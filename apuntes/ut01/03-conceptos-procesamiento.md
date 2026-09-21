```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción al Big Data
Apartado:                   3.- Conceptos de procesamiento
Resultados de aprendizaje:  ?
```


# UT01. INTRODUCCIÓN AL BIG DATA



## 3. Conceptos de procesamiento

### 3.1. ¿Qué es el procesamiento en Big Data?

Se define como el conjunto de técnicas, modelos, algoritmos y herramientas tecnológicas dirigidas a transformar grandes volúmenes de datos brutos en información estructurada, útil, interpretable y directamente aplicable al negocio. Dado el tamaño masivo de los datos, el procesamiento se ejecuta obligatoriamente de forma **distribuida**, repartiendo las tareas de cómputo en paralelo entre múltiples nodos del clúster.

El ciclo de procesamiento persigue un doble propósito estructurado en dos grandes bloques:

- **Gestión de los datos**: incluye tareas de obtención de los datos en bruto de diferentes fuentes, limpieza de los datos e integración y fusión de fuentes.
- **Análisis de los datos**: comprende el modelado y análisis estadístico y la posterior interpretación y visualización.



### 3.2. Tipos de procesamiento: Batch vs. Stream Processing

Ambas modalidades pueden coexistir dentro de una misma arquitectura para atender necesidades analíticas complementarias:

| Criterio | Procesamiento por Lotes (*Batch Processing*) | Procesamiento en Tiempo Real (*Stream Processing*) |
| -------- | -------------------------------------------- | --- |
| **Entrada de datos** | Conjuntos de datos completos acumulados durante un periodo. | Flujo continuo de eventos procesados mensaje a mensaje. |
| **Latencia**         | Minutos, horas o días. | Milisegundos o segundos. |
| **Prioridad**        | Integridad del cómputo y manejo de volúmenes masivos. | Inmediatez de respuesta y mínima latencia. |
| **Casos de uso**     | Cierre de facturación, consolidación contable, BI histórico, informes periódicos. | Detección de fraude, ajuste dinámico de precios, monitorización IoT, alertas de tráfico. |


### 3.3. Fases del procesamiento en Big Data

El flujo de procesamiento no se ejecuta en un único paso, sino mediante una secuencia estructurada de etapas independientes:

1. **Ingesta:** Captura de datos desde las fuentes generadoras (bases de datos relacionales, sensores IoT, APIs, redes sociales, archivos planos). Se realiza mediante conexiones directas, mecanismos de captura de cambios (*Change Data Capture - CDC*) o plataformas de mensajería y flujo continuo como *Apache Kafka* o *Apache NiFi*.
2. **Almacenamiento:** Depósito persistente o temporal en infraestructuras distribuidas y tolerantes a fallos, como Data Lakes en la nube (*Amazon S3*, *GCS*), sistemas de ficheros (*HDFS*, *MinIO*), bases de datos NoSQL (*MongoDB*, *Redis*, *Neo4j*) o Data Warehouses (*BigQuery*, *Snowflake*).
3. **Procesamiento y transformación (ETL / ELT):** Limpieza de anomalías, normalización de formatos, filtrado, reestructuración y aplicación de reglas de negocio (segmentación de registros, cálculo de métricas clave o cruces entre tablas).
4. **Análisis y modelado:** Aplicación de consultas analíticas interactivas (vía SQL sobre *Trino* o *Presto*), pipelines programados (*Spark*, *Flink*) o modelos avanzados de Machine Learning (como algoritmos de regresión, detección de patrones o pronóstico temporal mediante librerías como *Scikit-Learn* o *Prophet*).
5. **Visualización y puesta en valor:** Publicación de resultados e integración en la operativa diaria a través de cuadros de mando (*Tableau*, *Power BI*, *Superset*), APIs para aplicaciones empresariales o activación de alertas automáticas para la toma de decisiones.



---

[Volver al índice](ut01_index.md)