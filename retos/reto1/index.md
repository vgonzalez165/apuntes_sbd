```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. Persistencia documental, caché y procesamiento ETL
Reto:                       1. Sistema de triaje y trazabilidad de urgencias hospitalarias
Resultados de aprendizaje:  RA1, RA3
Criterios de evaluación:    RA1 c, RA1 d, RA1 f, RA3 a, RA3 b y RA3 d
```

# Reto 1: SISTEMA DE TRIAJE Y TRAZABILIDAD DE URGENCIAS HOSPITALARIAS

## Enunciado

[Enunciado del reto](./enunciado_alumno/index.md)

## Evaluación

[Criterios de evaluación del reto](./evaluación/index.md)

## Calendario

Aunque en el enunciado se expone en más detalle, en la siguiente tablas tienes un resumen de la planificación que debes seguir para abordar este reto.

| **Semana** | **Etapa**    | **Fecha**    | **Tareas de esa semana**                                                                                              |
| ---------- | ------------ | ------------ | --------------------------------------------------------------------------------------------------------------------- |
| **0**   | Presentación | `01-10-2026` | Repositorio creado. Kanban creado. Calendario de roles creado.                                                        |
| **1**   | Sprint 1     | `08-10-2026` | Despliegue de infraestructura en el servidor (fichero `compose.yml` único). Carga de datos. Conectividad desde Python |
| **2**   | Sprint 1     | `15-10-2026` | Limpieza de los datos. Tratamiento de nulos. Valores atípicos. Cruce de datos                                         |
| **3**   | Sprint 2     | `22-10-2026` | Carga de histórico en MongoDB. Creación de consultas de agregación                                                    |
| **4**   | Sprint 2     | `29-10-2026` | Envío de datos a Redis. Funciones para encolar pacientes y asignación de boxes.                                       |
| **5**   | Sprint 3     | `05-11-2026` | Conexión con la API REST. Implementación del bucle de sondeo y procesamiento lista altas e ingresos                   |
| **6**   | Sprint 3     | `12-11-2026` | Integración completa. Prueba de estrés. Incorporación del módulo opcional de anonimización                            |
| **7**   | Evaluación   | `19-11-2026` | Defensa del proyecto                                                                                                  |


## Contenidos asociados a este reto

| **Contenidos**                                                         | **Notebook de Jupyter**                           |
| ---------------------------------------------------------------------- | ------------------------------------------------- |
| [1. Introducción a Python Pandas](./apuntes/01_introduccion_pandas.md) | [`ipynb`](./apuntes/01_introduccion_pandas.ipynb) |
| [2. Carga de datos con Pandas](./apuntes/02_carga_datos_pandas.md)     | [`ZIP`](./apuntes/02_carga_datos_pandas.zip)      |
| [3. Limpieza de datos con Pandas](./apuntes/03_limpieza_datos.md)      | [`ipynb`](./apuntes/03_limpieza_datos.ipynb)      |
| [4. Anonimización de datos]()                                          |                                                   |
| [5. Bases de datos documentales (MongoDB)]()                           |                                                   |
| [6. Bases de datos clave-valor (Redis)]()                              |                                                   |