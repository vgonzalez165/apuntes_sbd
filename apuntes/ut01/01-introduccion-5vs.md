```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción al Big Data
Apartado:                   1.- Introducción. Las 5 V's del Big Data
Resultados de aprendizaje:  ?
```


# UT01. INTRODUCCIÓN AL BIG DATA


## 1. Introducción y las 5 V's del Big Data

### 1.1. ¿Qué es Big Data?

El término **Big Data** hace referencia a aquellos conjuntos de datos cuyo volumen, complejidad y velocidad de crecimiento hacen que las herramientas tradicionales de procesamiento de datos resulten insuficientes para capturarlos, almacenarlos, gestionarlos y analizarlos adecuadamente.

El ecosistema de Big Data procesa información proveniente de múltiples naturalezas y fuentes, clasificándose principalmente en tres tipologías de datos:

- **Estructurados:** Datos con un formato y esquema rígido predefinido, organizados habitualmente en filas y columnas como en las bases de datos relacionales.
- **Semiestructurados:** Información que no se ajusta rígidamente a una tabla formal pero contiene marcadores, etiquetas o jerarquías internas que facilitan su análisis, como los documentos en formatos JSON o XML y los archivos de registro (*logs*).
- **No estructurados:** Datos que carecen por completo de una estructura interna identificable de manera directa, tales como texto libre, correos electrónicos, documentos ofimáticos, archivos multimedia (imágenes, audios, vídeos), señales brutas de sensores o páginas web.

El propósito esencial de la disciplina reside en transformar estos grandes volúmenes de datos heterogéneos en información valiosa, permitiendo a las organizaciones mejorar la toma de decisiones, optimizar procesos operativos, diseñar nuevos productos o servicios y resolver problemas analíticos complejos.

### 1.2. El origen del concepto y el paradigma de las 5 V's

La caracterización del fenómeno Big Data mediante dimensiones que comienzan por la letra **V** se inició a principios de los años 2000, siendo el analista **Doug Laney** el primero en formalizar este enfoque. Inicialmente estructurado en torno a tres dimensiones fundamentales, el paradigma clásico se consolidó en las denominadas **5 V's**, que son volumen, velocidad, variedad, veracidad y valor.

#### A. Volumen

Representa la inmensa cantidad de información generada y almacenada a escala global de manera continua. Este crecimiento exponencial viene impulsado por:

- La proliferación masiva de **dispositivos conectados e Internet de las Cosas (IoT)**.
- La actividad continua en **redes sociales**, donde plataformas como Facebook, Instagram o X generan petabytes diarios.
- Las **transacciones digitales bancarias** y de **comercio electrónico**.
- Los datos generados automáticamente por **máquinas** (telemetría de servidores, sensores industriales y registros).

Este incremento ha desplazado las escalas habituales de medida hacia magnitudes como petabytes, exabytes y zettabytes. Según estudios del sector (como *Data Never Sleeps* de Domo y estimaciones de Statista), en 2022 el volumen global de datos generados y consumidos alcanzó los 97 zettabytes, proyectando alcanzar los 181 zettabytes en 2025 impulsado por una población conectada superior a los 5.000 millones de personas.

#### B. Velocidad

Describe la rapidez con la que los datos se generan, transmiten, procesan y analizan. En muchos escenarios críticos el valor del dato radica en la capacidad de respuesta inmediata:

- **Trading algorítmico y finanzas:** Ejecución de órdenes de compra/venta en milisegundos analizando variaciones de mercado.
- **Vehículos autónomos:** Procesamiento continuo y en fracciones de segundo de señales procedentes de cámaras, radares y sensores LIDAR.
- **Ciberseguridad:** Detección de patrones anómalos o ataques distribuidos (DDoS) al instante para mitigar daños.
- **Plataformas de streaming y redes sociales:** Ajuste dinámico de calidad de emisión y recomendación de contenidos en tiempo real según el comportamiento del usuario.

Para dar soporte a estas exigencias se emplean motores de flujo distribuido en tiempo real (*Apache Kafka*, *Apache Flink*, *Apache Storm*, *Spark Streaming*), bases de datos en memoria (*Redis*) y plataformas de análisis masivo altamente escalables (*Google BigQuery*, *Amazon Redshift*).

#### C. Variedad

Alude a la coexistencia y convivencia de múltiples formatos y modelos de datos:

- **Datos estructurados:** Almacenados en esquemas tabulares definidos con tipos de datos estrictos, consultables de forma óptima mediante SQL.
- **Datos semiestructurados:** Representados mediante formatos flexibles como JSON (estándar en APIs web y bases de datos documentales NoSQL), XML (empleado en intercambio empresarial entre sistemas heterogéneos) o ficheros de log de servidores con patrones reconocibles pero sin estructura tabular.
- **Datos no estructurados:** Archivos binarios, imágenes, señales continuas de sensores IoT, páginas HTML o texto plano en lenguaje natural, cuya explotación requiere técnicas avanzadas de procesamiento de lenguaje natural (PLN) o visión artificial.

La gestión de esta variedad impone desafíos específicos, como la necesidad de arquitecturas de almacenamiento híbridas y flexibles (HDFS, repositorios NoSQL), canalizaciones avanzadas de extracción, transformación y carga (ETL), mecanismos de control de calidad sin esquemas rígidos y la formación de perfiles técnicos multidisciplinares.

#### D. Veracidad

Define el nivel de fiabilidad, precisión, calidad y autenticidad que poseen los datos. Trabajar con datos de baja veracidad genera decisiones erróneas a nivel directivo y sesga los modelos de Inteligencia Artificial y Machine Learning.

Entre las causas más frecuentes de pérdida de veracidad destacan:

- Registros duplicados o con valores nulos.
- Muestras sesgadas o no representativas de la realidad.
- Fuentes de datos abiertas o de redes sociales sin validar.
- Ruido e interferencias en mediciones de sensores.

Para mitigar estos problemas se aplican estrategias de depuración e imputación de valores faltantes, validación cruzada entre fuentes fiables, monitorización continua de métricas de calidad y tecnologías avanzadas de trazabilidad como *blockchain* o algoritmos de detección de anomalías mediante IA.

#### E. Valor

Es considerada la dimensión más crítica. Consiste en la capacidad de transformar los datos brutos en conocimiento accionable y útil para la organización. Almacenar datos sin un propósito analítico claro genera un coste en infraestructura sin retorno de inversión.

Este principio se fundamenta en la cadena del conocimiento (modelo DIKW), que establece una pirámide con los siguientes elementos (comenzando por la base):

- **Datos**: Elementos discontinuos que representan hechos brutos.
- **Información**: Datos procesados y contextualizados para ser útiles.
- **Conocimiento**: Aplicación mental de los datos y la información.
- **Sabiduría**: Evaluación e internalización del conocimiento.

El valor se materializa cuando las organizaciones utilizan este conocimiento para enriquecer la experiencia de usuario (mediante motores de recomendación), optimizar costes de transporte y logística, o transformar el modelo de negocio con servicios basados en datos (como la fijación dinámica de precios y la asignación inteligente de rutas en empresas como Uber).



### 1.3. Otras V's del Big Data

Con la evolución de la tecnología se han incorporado dimensiones complementarias al modelo clásico:

- **Viabilidad:** Evaluación de la factibilidad económica y técnica del proyecto de datos (relación entre el coste del almacenamiento/cómputo y el retorno de inversión esperado).
- **Vulnerabilidad:** Riesgos vinculados a la privacidad de la información, cumplimiento normativo, fugas de datos e incidentes de seguridad en infraestructuras distribuidas.
- **Volatilidad:** Periodo de validez o ciclo de vida del dato; determina cuándo una información deja de ser útil o relevante para el negocio (por ejemplo, tendencias inmediatas en redes sociales).
- **Visualización:** Capacidad para exponer conjuntos complejos de información de manera gráfica, clara e intuitiva para facilitar su interpretación por perfiles no técnicos (utilizando herramientas como Tableau, Power BI o librerías de Python como Matplotlib).
- **Vinculación:** Capacidad de cruzar e integrar conjuntos de datos dispares e inconexos para enriquecer las conclusiones (por ejemplo, correlacionar series climáticas con el histórico de ventas de un producto).
- **Variabilidad:** Variación en el significado, formato o interpretación de los datos en función del contexto temporal, cultural o semántico.
- **Virtualización:** Abstracción del hardware físico mediante el uso de computación en la nube, contenedores y arquitecturas *serverless* (como AWS Lambda) para desacoplar el procesamiento del aprovisionamiento de máquinas.





---

[Volver al índice](ut01_index.md)






