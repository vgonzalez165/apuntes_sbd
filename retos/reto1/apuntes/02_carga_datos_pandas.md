```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. PERSISTENCIA DOCUMENTAL, CACHÉ Y PROCESAMIENTO ETL
Reto:                       1. Sistema de triaje y trazabilidad de urgencias hospitalarias
Apartado:                   2. Carga de datos con Pandas
Resultados de aprendizaje:  `RA1`, `RA3`
```


# 1.- Carga de datos con Python Pandas

## 2.1.- Archivos delimitados (CSV/TSV)

Aunque comúnmente llamados CSV (_Comma Separated Values_), son archivos de texto delimitados. Poseen tres componentes críticos:
- **Delimitador:** coma (`,`), punto y coma (`;`, habitual en Europa donde la coma es separador decimal), tabulador (`\t`, seguro frente a colisiones), barra vertical/pipe (`|`), secuencias multicarácter (`::`, `~!~`), espacios variables (logs) o caracteres ASCII no imprimibles de control (`\x01` en Apache Hive).
- **Codificación (_Encoding_):** `UTF-8` (estándar global compatible con caracteres internacionales y emojis) o `Latin-1` / `ISO-8859-1` / `cp1252` (habitual en entornos legacy y Excel tradicional).
- **Encabezado (_Header_):** presencia o ausencia de la fila descriptiva con los nombres de columnas.


```python
import io
import pandas as pd

# Simulación de un archivo CSV con inconsistencias
csv_data = """id;nombre;departamento;salario;fecha_alta
1001;Ana García;Marketing;45000;2021-03-15
1002;Carlos Ruiz;Ventas;38500;2022-07-01
1003;Elena Vázquez;IT;52000;2020-11-20
1004;Jorge Méndez;RRHH;ERROR;2023-01-10
1005;Lucía Torres;Finanzas;47000;2019-05-25
1006;Fila Invalida;Extra;Campo;Inesperado;SobranColumnas
1007;Marta Gil;Ventas;SIN_DATO;2024-02-01
"""

df_csv = pd.read_csv(
    io.StringIO(csv_data),  # Nombre del fichero (aquí usamos io.StringIO para simularlo)
    sep=';',                # Indicamos el separador del fichero
    header=0,               # Número de fila que contiene las columnas
    index_col='id',         # Qué columna será el índice
    na_values=['ERROR', 'SIN_DATO', '?', '-'],  # Estos valores los convertirá a NaN/None
    on_bad_lines='warn',    # Si hay una línea con errores avisa, pero sigue leyendo
                            # La línea con id 1006 (line 7) mostará un aviso
    engine='python',        # Motor de lectura, siempre pondremos este
    parse_dates=['fecha_alta'] # Activa la conversión de fechas en esta columna
)

print("DataFrame leído:")
print(df_csv)
print("\nTipos inferidos:") # Ya veremos como asignar el tipo correcto a cada columna
print(df_csv.dtypes)
```

    DataFrame leído:
                 nombre departamento  salario fecha_alta
    id                                                  
    1001     Ana García    Marketing  45000.0 2021-03-15
    1002    Carlos Ruiz       Ventas  38500.0 2022-07-01
    1003  Elena Vázquez           IT  52000.0 2020-11-20
    1004   Jorge Méndez         RRHH      NaN 2023-01-10
    1005   Lucía Torres     Finanzas  47000.0 2019-05-25
    1007      Marta Gil       Ventas      NaN 2024-02-01
    
    Tipos inferidos:
    nombre                  object
    departamento            object
    salario                float64
    fecha_alta      datetime64[ns]
    dtype: object


    /tmp/ipykernel_738/3601166916.py:15: ParserWarning: Skipping line 7: Expected 5 fields in line 7, saw 6
    
      df_csv = pd.read_csv(


## 2.2 Hojas de Cálculo (Excel)

A diferencia del texto plano, un fichero `.xlsx` es un contenedor comprimido (ZIP) compuesto por archivos XML, estilos, metadatos y varias hojas, lo que demanda librerías auxiliares como `openpyxl` (para `.xlsx`) o `xlrd` (para formatos antiguos `.xls`).


```python
import pandas as pd

# Ejemplo 1: Carga de una hoja concreta saltando filas decorativas (títulos/logos)
df_excel = pd.read_excel(
    'nomina_rrhh.xlsx', # Nombre del fichero
    sheet_name='Plantilla_Activa', # Nombre de la hoja
    engine='openpyxl',  # Motor de lectura
    header=2,           # Encabezados reales ubicados en la fila 3
    usecols='A:E',      # Limitar lectura a columnas de interés
    skipfooter=2,       # Omitir filas de totales al pie
    dtype={'ID_Empleado': str}
)

# Ejemplo 2: Carga de todas las hojas en un diccionario de DataFrames (ojo que devuelve un diccionario)
dict_hojas = pd.read_excel('datos_anuales.xlsx', sheet_name=None)
print("Hojas disponibles:", list(dict_hojas.keys()))
```

## 2.3 Formato JSON y JSON Lines

JSON es el estándar dominante en APIs web y bases de datos documentales. Su estructura jerárquica y anidada requiere técnicas de aplanado (_flattening_) para representarse de forma tabular.
- `pd.json_normalize()`: Aplana diccionarios anidados utilizando notación de puntos para las columnas compuestas.
- `record_path` y `meta`: Permiten desplegar listas internas como filas individuales, conservando atributos del nodo padre.
- **JSON Lines (`.jsonl`):** Un objeto JSON completo por línea. Muy eficiente para logs y streaming. Requiere `lines=True`.


```python
import json
import pandas as pd

# Estructura jerárquica con listas anidadas
raw_json = [
    {
        "ciclo": "DAM",
        "curso": "Primero",
        "estudiantes": [
            {"nombre": "Juan", "nota": 8},
            {"nombre": "Lucía", "nota": 9}
        ]
    },
    {
        "ciclo": "ASIR",
        "curso": "Segundo",
        "estudiantes": [
            {"nombre": "Pedro", "nota": 5}
        ]
    }
]

# Normalización: cada elemento de 'estudiantes' es una fila; 'ciclo' y 'curso' se replican
df_alumnos = pd.json_normalize(
    raw_json,
    record_path=['estudiantes'],  # Lista que quieres desglosar en filas, cada elemento una fila
    meta=['ciclo', 'curso']       # Estos datos se repetirán en todas las filas
)
print("Datos normalizados:")
print(df_alumnos)
```

    Datos normalizados:
      nombre  nota ciclo    curso
    0   Juan     8   DAM  Primero
    1  Lucía     9   DAM  Primero
    2  Pedro     5  ASIR  Segundo


## 2.4 Formato XML

Formato jerárquico basado en etiquetas que requiere de la librería `lxml`. Se procesa mediante `pd.read_xml()`, navegando la estructura con expresiones **XPath**:

- **Rutas absolutas (`/raiz/nodo`):** recorrido estricto desde la raíz.
- **Rutas relativas (`//nodo`):** localiza el elemento en cualquier profundidad del árbol.
- **Predicados (`//nodo[@attr="valor"]`):** filtra nodos bajo condiciones de atributos específicas.


```python
import pandas as pd
import io

xml_content = """<?xml version="1.0" encoding="UTF-8"?>
<tienda>
    <seccion nombre="electronica">
        <producto id="A001">
            <nombre>Portátil</nombre>
            <precio>850.00</precio>
        </producto>
        <producto id="A002">
            <nombre>Ratón</nombre>
            <precio>25.00</precio>
        </producto>
    </seccion>
    <seccion nombre="hogar">
        <producto id="B001">
            <nombre>Lámpara</nombre>
            <precio>40.00</precio>
        </producto>
    </seccion>
</tienda>
"""

# Extracción únicamente de los productos pertenecientes a 'electronica'
df_xml = pd.read_xml(
    io.StringIO(xml_content),
    xpath='//seccion[@nombre="electronica"]/producto'
)
print(df_xml)
```

         id    nombre  precio
    0  A001  Portátil   850.0
    1  A002     Ratón    25.0


## 2.5 Ingesta desde APIs REST

Una API REST utiliza HTTP para interactuar con recursos identificados mediante URLs.
- **Métodos principales:** `GET` (lectura), `POST` (creación), `PUT` (actualización), `DELETE` (eliminación).
- **Códigos de estado:** `200` (OK), `400` (Bad Request), `401` (Unauthorized), `404` (Not Found), `500` (Internal Server Error).
- **Mecanismos de autenticación:**
    - **API Key:** Token enviado en la _Query String_ de la URL (`params={...}`) o en cabeceras.
    - **Bearer Token:** Token criptográfico (habitual en flujos OAuth / JWT) sin estado enviado en la cabecera `Authorization: Bearer <token>`.
    - **Basic Auth:** Envío codificado de usuario y contraseña.


```python
import pandas as pd
import requests

# 1. Hacemos la petición a la API de Star Wars
url = "https://swapi.dev/api/planets"  # URL del endpoint que leemos
response = requests.get(url)

# 2. Convertimos la respuesta a formato JSON (diccionario de Python)
data = response.json()

# 3. Aplanamos la estructura usando json_normalize
# SWAPI devuelve los datos dentro de una lista llamada 'results'
df_planets = pd.json_normalize(data, record_path=["results"])

# 4. Seleccionamos y mostramos algunas columnas relevantes
columnas_interes = ["name", "diameter", "climate", "terrain"]
print(df_planets[columnas_interes].head())
```

           name diameter              climate                             terrain
    0  Tatooine    10465                 arid                              desert
    1  Alderaan    12500            temperate               grasslands, mountains
    2  Yavin IV    10200  temperate, tropical                 jungle, rainforests
    3      Hoth     7200               frozen  tundra, ice caves, mountain ranges
    4   Dagobah     8900                murky                      swamp, jungles



```python
import requests
import pandas as pd
from requests.auth import HTTPBasicAuth

# 1. Petición con Query String (API Key)
url_api = "https://api.openweathermap.org/data/2.5/weather"
parametros = {
    "lat": 42.60,
    "lon": -5.57,
    "appid": "TU_API_KEY",
    "units": "metric"
}
# resp = requests.get(url_api, params=parametros)

# 2. Petición con Bearer Token en cabeceras HTTP
headers = {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsIn...",
    "Accept": "application/json"
}
# resp_auth = requests.get("https://api.themoviedb.org/3/movie/popular", headers=headers)

# 3. Autenticación Básica (Legacy / Jira)
# resp_basic = requests.get("https://servidor.empresa.com/api", auth=HTTPBasicAuth('usuario', 'password'))

# Procesamiento del cuerpo JSON a DataFrame
datos_simulados = {
    "page": 1,
    "results": [
        {"id": 101, "titulo": "Película A", "valoracion": 8.4},
        {"id": 102, "titulo": "Película B", "valoracion": 7.1}
    ]
}
df_api = pd.DataFrame(datos_simulados["results"])
print(df_api)
```
```

### 2.6 Extracción Web (Web Scraping y `read_html`)

- **Técnica:** Obtención programática de información contenida en sitios web.
- **Retos habituales:** Carga de contenido dinámico mediante JavaScript, volatilidad del DOM, bloqueos de IP, resolución de CAPTCHAs y necesidad de respetar `robots.txt` y cabeceras `User-Agent`.
- **Tablas directas:** `pd.read_html()` localiza y procesa etiquetas `<table>` convirtiéndolas en DataFrames de forma automática utilizando como motor `lxml` o `BeautifulSoup`.


```python
# Pendiente de hacer
```

## 2.6 Almacenamiento de objetos compatible con S3 (AWS Academy y SeaweedFS)

En arquitecturas Big Data y *Data Lakes*, el almacenamiento masivo desacoplado del cómputo no utiliza sistemas de ficheros tradicionales (POSIX), sino **almacenes de objetos**. La API de **Amazon S3** se ha convertido en el estándar *de facto* de la industria, adoptado tanto por proveedores cloud como por soluciones locales de código abierto. 

Algunos conceptos clave que tenemos que conocer cuando trabajamos con S3:

- **Bucket y Clave (Key):** un bucket es un contenedor raíz plano y una clave (`datos/raw/salarios.csv`) es el identificador único del objeto. Las carpetas no existen físicamente en S3; son prefijos simulados por la barra (`/`).
- **Librería `s3fs`:** Pandas delega las conexiones con protocolo `s3://` en la librería `s3fs`. Debe estar instalada en el entorno (`pip install s3fs`).
- **AWS S3 (AWS Academy):** En entornos educativos como AWS Academy Learner Lab, las cuentas utilizan credenciales temporales generadas mediante AWS STS. Por ello, además de `key` y `secret`, es obligatorio suministrar el parámetro `token` (`aws_session_token`), el cual caduca periódicamente.


- **SeaweedFS (On-Premise / Laboratorio local):** Es un sistema de almacenamiento distribuido de alto rendimiento que incluye una capa de compatibilidad S3 (`weed s3`). Para conectarse a él es imprescindible redirigir las peticiones mediante `endpoint_url` y habilitar el direccionamiento por ruta (`addressing_style: 'path'`) para evitar que el cliente intente resolver subdominios DNS locales que no existen.


```python
import pandas as pd

# ==============================================================================
# CONFIGURACIÓN DE CONEXIÓN VÍA storage_options
# ==============================================================================

# ESCENARIO A: AWS Academy Learner Lab (Cloud AWS)
# En este caso necesitaremos obtener las credenciales desde la consola de AWS Academy. Recuerda que estas se encuentran en AWS Details
storage_options_aws = {
    "key": "ASIAXXXXXXXXXXXXXXXX",               # aws_access_key_id
    "secret": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY", # aws_secret_access_key
    "token": "IQoJb3JpZ2luX2VjE...",             # aws_session_token (Obligatorio en AWS Academy. Caduca cada vez que cambiamos de laboratorio)
    "client_kwargs": {
        "region_name": "us-east-1"               # Región por defecto en AWS Academy
    }
}

# ESCENARIO B: Cluster local con SeaweedFS
# El servicio S3 de SeaweedFS corre habitualmente en el puerto 8333
storage_options_seaweed = {
    "key": "alumno",                              # Configurado en s3.json de SeaweedFS
    "secret": "paso",
    "client_kwargs": {
        # IP o DNS del nodo SeaweedFS y puerto donde corre el servicio S3
        "endpoint_url": "http://10.201.59.248:8333" 
    },
    "config_kwargs": {
        "s3": {
            # Obligatorio en despliegues locales: fuerza peticiones http://ip:puerto/bucket/objeto
            # en lugar de http://bucket.ip:puerto/objeto
            "addressing_style": "path"
        }
    }
}

# ==============================================================================
# LECTURA DEL FICHERO DESDE PANDAS
# ==============================================================================

# Seleccionamos las opciones deseadas (cambiar entre seaweedfs y aws según la práctica)
opciones_activas = storage_options_seaweed

# La URI mantiene el formato estándar s3://<bucket>/<clave>
s3_uri = "s3://empresa-datalake-landing/rrhh/salarios.csv"

# Lectura directa: Pandas descarga el stream en memoria y procesa los delimitadores
df_s3 = pd.read_csv(
    s3_uri,
    storage_options=opciones_activas,            # Inyecta credenciales y endpoints a s3fs
    sep=';',                                     # Mantenemos las opciones de limpieza vistas en 1.1
    header=0,
    index_col='id',
    na_values=['ERROR', 'SIN_DATO', '?', '-'],
    on_bad_lines='warn',
    engine='python',
    parse_dates=['fecha_alta']
)

print("DataFrame leído correctamente desde almacenamiento compatible con S3:")
print(df_s3)
print("\nTipos inferidos:")
print(df_s3.dtypes)
```
