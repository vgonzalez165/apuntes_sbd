# Guía docente: Sprint 1 (Sistemas de Big Data)

- **Duración:** 6 semanas (12 horas presenciales a razón de 2 horas/semana).
- **Marco de trabajo:** CRISP-DM (Fases: *Business Understanding*, *Data Understanding*, *Data Preparation*, *Deployment* básico).
- **Mapeo curricular:**
  - **RA 1:** Criterios **c** (combinación de fuentes), **d** (conjuntos de datos complejos relacionados), **f** (selección e integración de sistemas NoSQL).
  - **RA 3:** Criterios **a** (extracción y almacenamiento de diversas fuentes), **b** (eficiencia en la extracción de valor), **d** (desarrollo de sistemas eficientes, seguros y contenerizados).



## Planificación y secuenciación didáctica (12 horas)

| **Semana** | **Fase**      | **Explicación**| **Trabajo**                                       | **Entregable** |
| ------ | ----------------- | -------------- | ------------------------------------------------- | ---------- |
| **1** | Comprensión de negocio | Contenedores   | Configuración entorno base y exploración datasets |  Repositorio Git inicial con stack Docker funcional y análisis exploratorio preliminar |
| **2** | Preparación de datos   | Técnicas de saneamiento. Optimización de tipos                     | Script ETL para normalizar fuentes sucias | Script `etl_cleaner.py` ejecutable y reporte de calidad del dato |
| **3** | Modelado de datos      | MongoDB        | Carga de datos en MongoDB y consultas             | Módulo de persistencia documental y tres consultas analíticas. |
| **4** | Modelado de datos      | Redis          | Lógica del sistema de triaje en tiempo real       | Módulo Redis para encolar pacientes y asignar boxes |
| **5** | Integración End-to-End | Conexión de componentes | Taller?                                  | Pipeline integrado ejecutable mediante script lanzador. |
| **6** | *Live Demo* por equipos|    |                                                               | Entrega final del repositorio, documentación técnica y defensa. |




| Semana / Sesión (2h) | Fase CRISP-DM y Contenidos técnicos | Dinámica en el aula | Entregable de sprint |
| ----------------- | --- | --- | --- |
| **Semana 1 (2h)** | *Business & Data Understanding*. Despliegue de infraestructura con `docker-compose` (MongoDB + Redis). | **Píldora (30 min):** Persistencia políglota y contenedores.<br
<br>**Taller (90 min):** Configuración del entorno base y exploración inicial de los datasets crudos. | Repositorio Git inicial con stack Docker funcional y análisis exploratorio preliminar. |
| **Semana 2 (2h)** | *Data Preparation*. Ingesta y limpieza masiva con Python Pandas. Tratamiento de nulos, tipos inconsistentes y parseo temporal. | **Píldora (20 min):** Técnicas de saneamiento y optimización de tipos con Pandas.
<br>**Taller (100 min):** Desarrollo del script ETL para normalizar las fuentes sucias. | Script `etl_cleaner.py` ejecutable y reporte de calidad del dato. |
| **Semana 3 (2h)** | *Data Modeling (Documental)*. Persistencia polimórfica en MongoDB. Operaciones de inserción masiva y pipelines de agregación. | **Píldora (25 min):** Documentos embebidos vs. referencias y agregaciones en MongoDB.
<br>**Taller (95 min):** Carga del dataset limpio en MongoDB y consultas de analítica clínica. | Módulo de persistencia documental y 3 consultas analíticas en `pymongo`. |
| **Semana 4 (2h)** | *Data Modeling (In-Memory)*. Colas de prioridad con Redis *Sorted Sets* y gestión de estado con *Hashes*. | **Píldora (25 min):** Complejidad algorítmica en Redis ($O(\log N)$) aplicada a colas de triage.
<br>**Taller (95 min):** Lógica del sistema de triage en tiempo real. | Módulo Redis para encolar pacientes y asignar boxes de atención. |
| **Semana 5 (2h)** | Integración *End-to-End* y pruebas de estrés. Simulación de flujo de pacientes. | **Taller guiado (120 min):** Conexión de componentes (ETL $\rightarrow$ MongoDB $\leftrightarrow$ Redis) y refactorización ante fallos. | Pipeline integrado ejecutable mediante script lanzador. |
| **Semana 6 (2h)** | Evaluación: *Live Demo* por equipos y prueba individual en máquina. | **Evaluación (60 min):** Prueba práctica individual.
<br>**Demos (60 min):** Defensa de 10 min por equipo frente al grupo. | Entrega final del repositorio, documentación técnica y defensa. |



## Estrategia de evaluación y ponderación

- **Producto técnico del equipo (40%):** evaluado mediante rúbrica sobre el repositorio Git. Criterios: contenerización limpia, robustez del ETL en Pandas, modelado documental justificado y uso eficiente de primitivas de Redis.
- **Prueba práctica individual en máquina (45%):** prueba cronometrada de 40 minutos en la semana 6. Cada alumno recibe un dataset ciego reducido y debe:
  1. Escribir una función en Pandas para filtrar/transformar registros bajo una condición específica.
  2. Construir una consulta de agregación en MongoDB que extraiga una métrica clínica.
  3. Insertar y recuperar un registro priorizado en Redis usando comandos nativos.
- **Defensa oral y coevaluación (15%):** Justificación de decisiones arquitectónicas durante la demo (10%) y factor de corrección de trabajo en equipo mediante rúbrica intra-equipo (5%).





**Enunciado para el Alumnado: Reto 1**

**Reto Técnico: Sistema de Triage y Trazabilidad de Urgencias Hospitalarias (*CareStream*)**

**1. Contexto del Reto**

El Servicio de Urgencias de la red hospitalaria sufre episodios recurrentes de congestión. Actualmente conviven dos problemas críticos:

1. **Pérdida de rendimiento en admisión:** Los historiales clínicos se reciben en formatos dispares (partes de ambulancias en JSON semiestructurado y registros de admisión en CSV con esquemas inconsistentes), provocando caídas en las bases relacionales tradicionales por bloqueos de tabla.
2. **Latencia en la priorización de pacientes:** El personal de triaje necesita determinar al milisegundo qué paciente debe entrar a box según la escala Manchester (Nivel 1: Resucitación inmediata $\rightarrow$ Nivel 5: No urgente), balanceando gravedad y tiempo de espera acumulado.

Vuestro equipo de ingeniería de datos debe diseñar, desplegar y validar un prototipo funcional que resuelva la ingesta políglota, almacene el histórico clínico y gestione la cola de espera en tiempo real.

---

**2. Requisitos Técnicos y Arquitectura del Sistema**

El sistema debe operar íntegramente de forma reproducible mediante Docker y componerse de los siguientes módulos:

```
                  ┌────────────────────────┐
                  │ Fuentes Crudas         │
                  │ (CSV admisión + JSON)  │
                  └───────────┬────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │ Script ETL (Pandas)    │
                  │ Limpieza y normalizado │
                  └───────┬────────┬───────┘
                          │        │
           (Histórico)    │        │  (Estado activo / Triage)
                          ▼        ▼
           ┌──────────────────┐  ┌──────────────────┐
           │     MongoDB      │  │      Redis       │
           │  (Episodios y    │  │ (Sorted Sets y   │
           │   anamnesis)     │  │  Hashes de boxes)│
           └──────────────────┘  └──────────────────┘

```

* **Infraestructura (`docker-compose.yml`):**
* Un servicio para **MongoDB** (con persistencia montada en volumen).
* Un servicio para **Redis**.
* Un contenedor opcional para la ejecución de scripts Python con dependencias (`pandas`, `pymongo`, `redis`).


* **Módulo ETL (`etl_pipeline.py`):**
* Ingesta de los dos datasets crudos suministrados.
* Detección y tratamiento explícito de: fechas en formatos incompatibles, duplicados en identificadores de paciente (`SIP`), valores de constantes vitales corruptos (ej. frecuencias cardíacas negativas) y valores nulos en campos clínicos críticos.
* Formateo de los datos depurados para su inserción masiva.


* **Persistencia Documental en MongoDB (`hospital_db`):**
* Colección `episodios_urgencias`: cada documento debe representar el paso completo de un paciente por el servicio, soportando atributos variables (un paciente traumatológico tiene campos de radiología que no existen en un paciente pediátrico).
* Implementar un script con **dos consultas analíticas de agregación**:
1. Tiempo medio de estancia en urgencias agrupado por patología de triaje.
2. Porcentaje de derivaciones a planta (ingreso hospitalario) vs. alta domiciliaria por grupo de edad.




* **Motor de Triage y Camas en Redis (`triage_engine.py`):**
* **Cola de Triage:** Uso de un *Sorted Set* (`urgencias:cola_espera`). El `score` numérico debe calcularse algorítmicamente ponderando el nivel de gravedad Manchester (1 a 5, donde 1 es máxima prioridad) y los minutos transcurridos desde la admisión.
* **Gestión de Boxes:** Uso de *Hashes* (`box:1`, `box:2`, etc.) para controlar en tiempo real qué paciente ocupa cada box, médico asignado y hora de entrada.
* Implementar funciones operativas mínimas:
* `admitir_paciente(id_paciente, nivel_manchester)`: Encola al paciente con su prioridad calculada.
* `llamar_siguiente_paciente(id_box)`: Extrae de forma atómica al paciente más prioritario y le asigna el box correspondiente.
* `liberar_box(id_box)`: Vía de salida que actualiza el historial en MongoDB y deja el box disponible.





---

**3. Fuentes de Datos Facilitadas**

En el repositorio base del reto encontraréis dos ficheros con datos sintéticos sucios:

1. `admisiones_historico.csv`: Contiene 50.000 registros con campos: `id_episodio`, `sip_paciente`, `timestamp_llegada`, `motivo_consulta`, `frecuencia_cardiaca`, `tension_arterial`, `destino_alta`.
2. `partes_clinicos.json`: Contiene 15.000 documentos semiestructurados con datos de constantes complementarias, antecedentes personales, alergias y notas médicas en texto libre.

---

**4. Hitos de Entrega y Criterios de Aceptación (*Definition of Done*)**

* **Hito 1 (Fin de Semana 2):** Archivo `docker-compose.yml` validado que levanta los servicios sin errores. Script de Pandas que procesa los dos archivos crudos, genera un informe con los registros descartados/corregidos y exporta los datos limpios.
* **Hito 2 (Fin de Semana 4):** Colección en MongoDB poblada mediante script automatizado con índices adecuados. Implementación de los scripts de Redis para encolar y desencolar pacientes según la prioridad algorítmica.
* **Hito 3 - Entrega Final (Fin de Semana 6):**
* Repositorio Git estructurado (`/docker`, `/src`, `/docs`).
* `README.md` exhaustivo con instrucciones exactas para ejecutar el pipeline de extremo a extremo con un único comando.
* Demostración en vivo de 10 minutos ante el aula simulando una llegada masiva de 20 pacientes con diferentes niveles de triaje y visualizando el vaciado correcto de la cola hacia los boxes.