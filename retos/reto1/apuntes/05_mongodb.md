```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. PERSISTENCIA DOCUMENTAL, CACHÉ Y PROCESAMIENTO ETL
Apartado:                   5.- MongoDB
Resultados de aprendizaje:  ?
```

# 5. Introducción a MongoDB con Python (`PyMongo`)

## 5.1. El paradigma NoSQL y las bases de datos documentales

En proyectos de Inteligencia Artificial y Big Data es habitual trabajar con datos semiestructurados (respuestas de APIs en formato JSON, logs de servidores, telemetría de sensores IoT o metadatos de entrenamientos) que cambian de estructura con frecuencia o crecen a gran velocidad. Las bases de datos relacionales (SQL) imponen esquemas rígidos y operaciones `JOIN` costosas al escalar en múltiples servidores.

**MongoDB** es una base de datos **NoSQL orientada a documentos** diseñada para resolver estos cuellos de botella mediante tres características clave de almacenamiento:

1. **Almacenamiento en formato BSON (Binary JSON):** Aunque interactuamos con los datos como si fueran objetos JSON o diccionarios de Python, internamente MongoDB los guarda en **BSON**, una representación binaria que:
   - Es mucho más rápida de recorrer e indexar que un archivo de texto plano.
   - Amplía los tipos de datos nativos de JSON añadiendo fechas (`Date`), identificadores únicos (`ObjectId`), datos binarios (`BinData`) y números de distinta precisión (`Int32`, `Int64`, `Decimal128`).
2. **Esquema flexible (*Schema-less* / Polimórfico):** Los documentos dentro de una misma colección no están obligados a tener exactamente las mismas claves ni los mismos tipos de datos. Puedes añadir nuevos campos a los registros nuevos sin necesidad de ejecutar un `ALTER TABLE` ni bloquear la base de datos.
3. **Localidad del dato (Desnormalización):** Mientras que en SQL se divide la información en múltiples tablas para evitar duplicidades, en MongoDB **los datos que se consultan juntos se almacenan juntos** (mediante listas y subdocumentos anidados). Esto permite recuperar toda la información de una entidad en una única lectura de disco, evitando cruces (`JOIN`) y facilitando el particionado horizontal (*Sharding*) entre varios nodos.

### Equivalencia conceptual: SQL vs. MongoDB vs. Python

| Mundo Relacional (SQL)         | Mundo MongoDB                                          | Equivalente en Python                   |
| ------------------------------ | ------------------------------------------------------ | --------------------------------------- |
| Base de Datos (*Database*)     | Base de Datos (*Database*)                             | Objeto `Database`                       |
| Tabla (*Table*)                | Colección (*Collection*)                               | Objeto `Collection`                     |
| Fila / Registro (*Row*)        | Documento BSON (*Document*)                            | Diccionario (`dict`)                    |
| Columna (*Column*)             | Campo / Clave (*Field*)                                | Clave del diccionario (`key`)           |
| Clave Primaria (`PRIMARY KEY`) | Campo `_id` (obligatorio, tipo `ObjectId` por defecto) | Clave `"_id"`                           |
| `JOIN` entre tablas            | Documentos embebidos (anidados) o etapa `$lookup`      | Diccionarios/listas dentro de un `dict` |


## 5.2. Conexión desde Python con `PyMongo`

Para conectar Python con MongoDB utilizamos la librería oficial `pymongo`. La gran ventaja para el desarrollo en Python es que **los documentos se manipulan directamente como diccionarios nativos (`dict`)**.

```bash
pip install pymongo pandas

```

### Creación perezosa (*Lazy Creation*)

En MongoDB no existe una sentencia obligatoria equivalente a `CREATE DATABASE` o `CREATE TABLE`. Tú referencias la base de datos y la colección en tu código, y MongoDB las creará físicamente en disco en el instante en que insertes el primer documento.

```python
from pymongo import MongoClient

# 1. Conectar al servidor (por defecto en localhost, puerto 27017)
client = MongoClient("mongodb://localhost:27017/")

# 2. Seleccionar (o referenciar para crear) la base de datos
db = client["ia_bigdata_db"]

# 3. Seleccionar (o referenciar para crear) la colección
coleccion_sensores = db["lecturas_iot"]

```

## 5.3. Operaciones CRUD básicas

Imagina que estamos almacenando lecturas de dispositivos IoT repartidos por una planta industrial para un posterior modelo de mantenimiento predictivo.

### 5.3.1. Inserción de datos (`Create`)

Si no especificamos el campo `"_id"`, MongoDB generará automáticamente un `ObjectId` único de 12 bytes (que incluye una marca temporal de creación).

```python
from datetime import datetime

# Insertar un único documento: insert_one()
lectura = {
    "sensor_id": "TEMP-01",
    "zona": "Sala Servidores",
    "temperatura": 24.5,
    "humedad": 45,
    "alertas": ["revision_anual"],
    "ubicacion": {"planta": 1, "rack": "A2"}, # Subdocumento anidado
    "fecha": datetime.now()
}

resultado = coleccion_sensores.insert_one(lectura)
print(f"Documento insertado con ID: {resultado.inserted_id}")

# Insertar múltiples documentos en bloque (mucho más eficiente en Big Data): insert_many()
lote_lecturas = [
    {"sensor_id": "TEMP-01", "zona": "Sala Servidores", "temperatura": 28.2, "humedad": 42, "estado": "OK"},
    {"sensor_id": "TEMP-02", "zona": "Almacén", "temperatura": 19.0, "humedad": 60, "estado": "OK"},
    {"sensor_id": "TEMP-03", "zona": "Sala Servidores", "temperatura": 35.8, "humedad": 38, "estado": "CRITICO"},
    {"sensor_id": "TEMP-02", "zona": "Almacén", "temperatura": 21.4, "humedad": 58, "estado": "OK"}
]

coleccion_sensores.insert_many(lote_lecturas)

```

### 5.3.2. Consultas básicas (`Read`)

Para consultar usamos `find_one()` (devuelve un único diccionario o `None`) y `find()` (devuelve un **cursor iterable**).

> **Nota importante de rendimiento:** `find()` no descarga todos los documentos de golpe a la memoria RAM de Python; devuelve un objeto `Cursor` que va pidiendo los datos al servidor a medida que lo recorres en un bucle o lo conviertes en lista.

```python
# 1. Obtener un solo documento que cumpla el filtro
doc = coleccion_sensores.find_one({"sensor_id": "TEMP-03"})
print(doc)

# 2. Obtener todos los documentos de una zona (filtro de igualdad)
cursor = coleccion_sensores.find({"zona": "Sala Servidores"})
for documento in cursor:
    print(documento)

```

#### Proyecciones (Seleccionar qué campos devolver)

En Ciencia de Datos no queremos traer campos pesados innecesarios por la red. El segundo argumento de `find(filtro, proyeccion)` indica con `1` los campos a incluir y con `0` los campos a excluir:

```python
# Equivalente a: SELECT sensor_id, temperatura FROM lecturas_iot WHERE zona = 'Sala Servidores'
# Nota: _id siempre se devuelve por defecto salvo que pongamos explícitamente "_id": 0
cursor = coleccion_sensores.find(
    {"zona": "Sala Servidores"},                  # Filtro (WHERE)
    {"_id": 0, "sensor_id": 1, "temperatura": 1}  # Proyección (SELECT)
)
print(list(cursor))

```

#### Operadores de comparación y lógicos

En MongoDB los operadores empiezan por el símbolo `$` y se expresan como un diccionario anidado dentro del campo a evaluar:

| Operador       | Significado                                   | Ejemplo de sintaxis                                              |
| -------------- | --------------------------------------------- | ---------------------------------------------------------------- |
| `$eq` / `$ne`  | Igual a / Distinto de                         | `{"estado": {"$ne": "OK"}}`                                      |
| `$gt` / `$gte` | Mayor que / Mayor o igual que                 | `{"temperatura": {"$gt": 25}}`                                   |
| `$lt` / `$lte` | Menor que / Menor o igual que                 | `{"humedad": {"$lte": 50}}`                                      |
| `$in` / `$nin` | Dentro de una lista / No en lista             | `{"sensor_id": {"$in": ["TEMP-01", "TEMP-02"]}}`                 |
| `$exists`      | Comprueba si el campo existe en el documento  | `{"alertas": {"$exists": True}}`                                 |
| `$or`          | Condición lógica OR (recibe lista de filtros) | `{"$or": [{"temperatura": {"$gt": 30}}, {"estado": "CRITICO"}]}` |

```python
# Ejemplo: Lecturas de "Sala Servidores" con temperatura superior a 25 grados (AND implícito)
filtro = {
    "zona": "Sala Servidores",
    "temperatura": {"$gt": 25}
}

# Ordenar por temperatura descendente (-1) y limitar a los 5 primeros
resultados = coleccion_sensores.find(filtro).sort("temperatura", -1).limit(5)

for r in resultados:
    print(r)

```

*Nota sobre subdocumentos:* Para filtrar por un campo dentro de un diccionario anidado se usa la **notación de punto** entre comillas: `{"ubicacion.planta": 1}`.

### 5.3.3. Actualización y Borrado (`Update` y `Delete`)

Para modificar documentos sin sobrescribirlos por completo se utilizan operadores de actualización como `$set` (asignar o crear un campo) o `$inc` (incrementar un valor numérico).

```python
# Actualizar un documento: cambiar el estado y añadir un campo "revisado"
coleccion_sensores.update_one(
    {"sensor_id": "TEMP-03", "estado": "CRITICO"},         # Filtro de búsqueda
    {"$set": {"estado": "EN_REVISION", "revisado": True}}  # Modificación a aplicar
)

# Incrementar en +0.5 la temperatura de todos los sensores del Almacén (calibración)
coleccion_sensores.update_many(
    {"zona": "Almacén"},
    {"$inc": {"temperatura": 0.5}}
)

# Eliminar documentos que cumplan una condición
coleccion_sensores.delete_many({"estado": "DESCARTADO"})

```


## 5.4. El Framework de Agregación (`aggregate`)

Cuando necesitamos agrupar datos, calcular estadísticas o transformar estructuras antes de entrenar un modelo, no es eficiente descargar millones de documentos a Python para procesarlos. En su lugar, usamos un **Pipeline de Agregación** que se ejecuta directamente en el motor de MongoDB (escrito en C++).

Un *pipeline* es una **lista de Python (`[...]`)** donde cada elemento es un diccionario que representa una **etapa** (*stage*). Los documentos entran en la primera etapa, se transforman o filtran, y el resultado pasa a la siguiente etapa como en una tubería de Linux.

### Etapas más utilizadas

1. **`$match`**: Filtra documentos (equivalente a `WHERE`). Se coloca siempre lo más arriba posible para reducir el volumen de datos a procesar en las siguientes etapas y aprovechar los índices.
2. **`$group`**: Agrupa documentos por un criterio obligatorio llamado `"_id"` (equivalente a `GROUP BY`) y permite aplicar acumuladores matemáticos:
    - `{"$sum": 1}` (contar elementos) o `{"$sum": "$campo"}` (sumar valores).
    - `{"$avg": "$campo"}` (media), `{"$max": "$campo"}` (máximo), `{"$min": "$campo"}` (mínimo).
    - *Importante:* Cuando referenciamos el valor de un campo existente a la derecha de los dos puntos, su nombre debe llevar el prefijo `$` (ej. `"$temperatura"`).
3. **`$project`**: Renombra, oculta o crea nuevos campos calculados en la salida.
4. **`$sort`** y **`$limit`**: Ordenan (`1` ascendente, `-1` descendente) y recortan los documentos resultantes.

### Ejemplo práctico de agregación en Python

Queremos obtener, solo para los sensores que tengan el campo `"temperatura"`, la **temperatura media, la máxima y el total de lecturas registradas por cada zona**, ordenadas de mayor a menor temperatura media:

```python
pipeline = [
    # Etapa 1: Filtrar solo documentos válidos con temperatura mayor a 0
    {
        "$match": {
            "temperatura": {"$gt": 0}
        }
    },
    # Etapa 2: Agrupar por el campo "zona" y calcular métricas
    {
        "$group": {
            "_id": "$zona",                                 # Campo de agrupación (GROUP BY zona)
            "temp_media": {"$avg": "$temperatura"},         # AVG(temperatura)
            "temp_maxima": {"$max": "$temperatura"},        # MAX(temperatura)
            "total_lecturas": {"$sum": 1}                   # COUNT(*)
        }
    },
    # Etapa 3: Ordenar por temperatura media de mayor a menor
    {
        "$sort": {
            "temp_media": -1
        }
    },
    # Etapa 4 (Opcional): Formatear la salida para que sea más limpia
    {
        "$project": {
            "_id": 0,
            "zona": "$_id",
            "temp_media": {"$round": ["$temp_media", 2]},   # Redondear a 2 decimales
            "temp_maxima": 1,
            "total_lecturas": 1
        }
    }
]

# Ejecutar el pipeline
resultado_agregacion = coleccion_sensores.aggregate(pipeline)

for fila in resultado_agregacion:
    print(fila)

```

**Salida esperada:**

```python
{'temp_maxima': 35.8, 'total_lecturas': 3, 'zona': 'Sala Servidores', 'temp_media': 29.5}
{'temp_maxima': 21.9, 'total_lecturas': 2, 'zona': 'Almacén', 'temp_media': 20.7}

```

## 5.5. Integración con Pandas 

Como tanto `find()` como `aggregate()` devuelven cursores de diccionarios de Python, transformarlos en un `DataFrame` de **Pandas** para análisis exploratorio o entrenamiento con `Scikit-Learn` es inmediato:

```python
import pandas as pd

# 1. Ejecutar consulta o agregación (excluyendo _id si no aporta valor al modelo)
cursor = coleccion_sensores.find(
    {"temperatura": {"$exists": True}},
    {"_id": 0, "sensor_id": 1, "zona": 1, "temperatura": 1, "humedad": 1}
)

# 2. Convertir el cursor en lista y pasarlo directamente a Pandas
df = pd.DataFrame(list(cursor))

print(df.head())

```