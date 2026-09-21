```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción al Big Data
Apartado:                   2.- Conceptos de almacenamiento
Resultados de aprendizaje:  ?
```


# UT01. INTRODUCCIÓN AL BIG DATA


## 2. Conceptos de almacenamiento

### 2.1. Sistemas de archivos distribuidos

Durante décadas la persistencia empresarial descansó exclusivamente en los Sistemas de Gestión de Bases de Datos Relacionales (RDBMS), los cuales garantizan transacciones seguras bajo esquemas tabulares rígidos. No obstante, ante el volumen masivo y la heterogeneidad del Big Data, los esquemas relacionales tradicionales presentan dos limitaciones críticas:

1. **Escalabilidad vertical limitada y costosa:** Ampliar CPU, memoria RAM o discos en un único servidor físico alcanza rápidamente techos tecnológicos y económicos.
2. **Latencia elevada en sistemas de almacenamiento en red compartidos:** Soluciones tradicionales como NFS o SAN sufren cuellos de botella severos cuando deben atender lecturas y escrituras concurrentes a gran escala.

La respuesta técnica a estas restricciones es el **almacenamiento distribuido**, donde los archivos se dividen en bloques homogéneos y se reparten entre múltiples servidores estándar (*nodos*) agrupados en un *clúster*.


#### Ventajas y desafíos del almacenamiento distribuido

- **Ventajas:** Permite escalabilidad horizontal casi ilimitada (añadiendo más nodos al sistema), alta disponibilidad ante la caída de servidores y mejora del rendimiento global mediante lecturas y escrituras paralelas.
- **Desafíos:** Complejidad en la gestión de la consistencia entre réplicas distribuidas y necesidad de minimizar la latencia de comunicación a través de la red interna.


#### Mecanismos de replicación

Para garantizar tolerancia a fallos, sistemas como HDFS o Cassandra replican automáticamente cada bloque en varios nodos y racks:

- **Replicación síncrona:** La escritura solo se da por completada cuando todos los nodos réplica confirman el guardado. Garantiza consistencia fuerte, pero incrementa sensiblemente la latencia.
- **Replicación asíncrona:** Confirma la operación de inmediato en el nodo primario y propaga la copia al resto de forma diferida. Ofrece mayor rendimiento a costa de tolerar una posible inconsistencia temporal.


#### Modelos de consistencia: ACID vs. BASE

- **Consistencia Fuerte (Modelo ACID):** Propio de los sistemas relacionales tradicionales. Asegura Atomicidad (las transacciones son todo o nada), Consistencia (se preserva la integridad de los datos), Aislamiento (las operaciones concurrentes no interfieren entre sí) y Durabilidad (los cambios persisten tras caídas). En entornos distribuidos masivos, sincronizar todos los nodos penaliza gravemente la latencia y la disponibilidad.
- **Consistencia Eventual (Modelo BASE):** Diseñado para sistemas NoSQL y distribuidos. Se define por *Basically Available* (el clúster siempre ofrece respuesta aunque sea degradada), *Soft state* (el estado del sistema puede fluctuar con el tiempo) y *Eventually Consistent* (con el tiempo suficiente sin nuevas escrituras, todos los nodos convergen al mismo estado). Sacrifica la consistencia inmediata a cambio de máxima disponibilidad y escalabilidad horizontal.


#### El Teorema CAP

Formulado por **Eric A. Brewer**, demuestra que un sistema distribuido que comparte datos solo puede garantizar de manera simultánea dos de las siguientes tres propiedades:

1. **Consistencia (C):** Cada lectura recibe el dato más reciente o genera un error.
2. **Disponibilidad (A):** Cada petición no errónea recibe una respuesta sin garantía de que contenga la versión más reciente.
3. **Tolerancia a Particiones (P):** El sistema continúa operando a pesar de que la red interna sufra caídas o pérdidas de mensajes entre nodos.


En función del compromiso de diseño adoptado, los sistemas se clasifican en:

- **Sistemas CP (Consistencia + Tolerancia a particiones):** Si ocurre una partición de red, el sistema bloquea operaciones para asegurar que los datos no diverjan. Son indispensables en registros financieros, transacciones críticas y control de configuraciones (ejemplos: *Apache HBase*, *Google Bigtable*, o *MongoDB* configurado con `writeConcern=majority`).
- **Sistemas AP (Disponibilidad + Tolerancia a particiones):** Si la red se fragmenta, todos los nodos siguen aceptando lecturas y escrituras aunque queden temporalmente desincronizados. Es el enfoque habitual en redes sociales, catálogos de comercio electrónico o analítica de flujos (ejemplos: *Apache Cassandra*, *Amazon DynamoDB*, *Project Voldemort*, *CouchDB*).
- **Sistemas CA (Consistencia + Disponibilidad):** Garantizan coherencia y respuesta continua, pero asumen que la red nunca se partirá. Por definición no pueden operar como sistemas distribuidos tolerantes a fallos de comunicación en red, siendo propios de arquitecturas centralizadas o servidores únicos (ejemplos: *PostgreSQL*, *MySQL*, *Redis* no clusterizado u *Oracle*).



### 2.2. Datasets

Un **dataset** es una colección de datos recopilados, organizados y estructurados (o no estructurados) que se utiliza como insumo para tareas de análisis, modelado algorítmico, exploración científica o toma de decisiones.

#### Características esenciales de los datasets en Big Data

- **Tamaño masivo:** Abarcan escalas que van desde gigabytes hasta petabytes o zettabytes.
- **Variedad de tipologías:** Integran datos tabulares, semiestructurados (JSON, XML) y contenidos no estructurados (imágenes, textos, audios).
- **Generación continua:** Flujos producidos en tiempo real o casi real procedentes de eventos de usuarios, telemetría IoT o registros de servidores.
- **Almacenamiento distribuido:** Persisten habitualmente sobre infraestructuras escalables como *Hadoop HDFS*, *Amazon S3* o *Google Cloud Storage*.


#### Clasificación según el uso

- **Operacionales:** Generados directamente por aplicaciones en producción y transacciones diarias (logs de sistemas, carritos de compra, mediciones de sensores).
- **Analíticos:** Datos que han sido extraídos, consolidados, limpiados y transformados para alimentar herramientas de Business Intelligence (BI), paneles de mando y modelos de pronóstico.
- **De entrenamiento:** Colecciones de datos preparadas específicamente para entrenar modelos de Inteligencia Artificial y Machine Learning, compuestas por vectores de características (*features*) y, en aprendizaje supervisado, sus etiquetas objetivo (*labels*).


#### Ejemplos de datasets abiertos de referencia

- **Twitter Stream API:** Flujo en tiempo real de publicaciones, menciones e interacciones públicas.
- **OpenStreetMap:** Base de datos geográfica y cartográfica colaborativa a escala global.
- **Common Crawl:** Repositorio masivo con miles de millones de páginas web rastreadas mensualmente.
- **ImageNet:** Banco de millones de imágenes etiquetadas manualmente, fundamental en el desarrollo de la visión por computador.
- **Netflix Prize Dataset:** Registros históricos anónimos de calificaciones de películas empleados para investigar algoritmos de recomendación.
- **San Francisco Open Data:** Datos públicos de gestión municipal (rutas de transporte, seguridad, urbanismo).
- **NOAA Climate Data Online:** Registros meteorológicos e históricos del clima con más de un siglo de antigüedad.
- **Human Connectome Project (HCP):** Imágenes cerebrales de alta resolución (resonancias magnéticas) y datos genéticos.


### 2.3. Data Warehouses, Data Lakes y Data Lakehouses

#### A. Data Warehouse

Es un repositorio de almacenamiento estructurado centralizado concebido para integrar y depurar información procedente de múltiples fuentes operacionales. Los datos se modelan y transforman **antes** de ser insertados (*Schema-on-write*), organizándose con frecuencia en áreas temáticas departamentales denominadas *Data Marts*. Resultan óptimos para consultas SQL analíticas, informes históricos y cuadros de mando corporativos (ejemplos: *Google BigQuery*, *Amazon Redshift*, *Microsoft SQL Server Analysis Services*).

Su arquitectura clásica comprende: Fuentes operacionales (ERP, CRM, ficheros) $\rightarrow$ Área de preparación o *Staging* (proceso ETL) $\rightarrow$ Almacén central (con metadatos y datos agregados) $\rightarrow$ *Data Marts* (ventas, finanzas, compras) $\rightarrow$ Consumo final por analistas.

#### B. Data Lake

Es un repositorio centralizado de alta capacidad que permite almacenar grandes volúmenes de datos en su formato nativo o en bruto (*raw data*), sin necesidad de transformarlos o definir un esquema previamente. Puede albergar simultáneamente datos estructurados, semiestructurados y no estructurados, difiriendo la estructura hasta el momento de la consulta (*Schema-on-read*). Ofrece gran flexibilidad y bajo coste, siendo el entorno idóneo para científicos de datos, ingenieros de machine learning y análisis exploratorio (ejemplos: *AWS Lake Formation*, *Apache HDFS*, *Azure Data Lake Storage*).

Su arquitectura contempla: Fuentes $\rightarrow$ Ingesta (por lotes o streaming) $\rightarrow$ Almacenamiento desacoplado (capas *Raw/Landing*, *Transform* y *Processed*) junto a entornos de experimentación (*Sandboxes*) $\rightarrow$ Consumo para analítica avanzada y BI.

#### C. Data Lakehouse

Representa el paradigma arquitectónico más reciente, combinando las principales ventajas de ambos mundos: la escalabilidad, bajo coste y flexibilidad con datos crudos del Data Lake, junto con las capacidades de gestión de transacciones, estructura y optimización de consultas SQL del Data Warehouse.

Sus características fundamentales incluyen:

- **Almacenamiento unificado:** Aloja datos estructurados, semiestructurados y no estructurados dentro del mismo repositorio.
- **Formatos de archivo abiertos y estandarizados:** Emplea estándares orientados a columnas como *Apache Parquet*, *ORC* o *Avro*, evitando el bloqueo de proveedor (*vendor lock-in*).
- **Desacoplamiento total de almacenamiento y cómputo:** Los datos se conservan en almacenamiento masivo económico (S3, GCS, HDFS) mientras que el cómputo se ejecuta mediante motores elásticos independientes (*Apache Spark*, *Trino*, *Presto*).
- **Soporte transaccional ACID:** Introduce consistencia transaccional, control de concurrencia y versionado mediante capas de metadatos avanzadas como *Delta Lake*, *Apache Iceberg* o *Apache Hudi*.
- **Cargas de trabajo mixtas:** Da servicio simultáneamente a procesos ETL por lotes, flujos en tiempo real (*Kafka*, *Flink*), consultas SQL interactivas y pipelines de aprendizaje profundo (*TensorFlow*, *PyTorch*).
- **Alto rendimiento y escalabilidad:** Aplica técnicas de indexación avanzada, almacenamiento columnar y sistemas de caché para optimizar el acceso a petabytes de información.


#### Capas de la arquitectura Data Lakehouse

1. **Fuentes de datos:** Orígenes transaccionales, APIs, ficheros JSON/XML, multimedia y flujos continuos.
2. **Capa de ingestión:** Procesos de captura tanto por lotes (*batch*) como en tiempo real (*streaming*).
3. **Capa de almacenamiento:** Repositorio Data Lake en formato abierto.
4. **Capa de metadatos y gobierno:** Control transaccional ACID, gestión de catálogos, políticas de seguridad, caché e indexación.
5. **Capa de APIs de acceso:** Exposición mediante interfaces SQL estándar, DataFrames declarativos y APIs de metadatos.
6. **Capa de consumo:** Cuadros de mando de BI (*Superset*, *LinceBI*), herramientas de exploración (*Jupyter*), motores de búsqueda (*Elasticsearch*) y modelos de *Data Science*.



---

[Volver al índice](ut01_index.md)