# 1. Limpieza de datos con Pandas

## 2.1. Datos de pruebas: creación del dataset "sucio"

Para comprender el alcance de los problemas habituales, utilizaremos un conjunto de datos que concentra las patologías más frecuentes de fuentes heterogéneas (APIs, logs desestructurados, volcados SQL sin restricciones):


```python
import numpy as np
import pandas as pd

# Desactiva el ajuste de línea automático para que use todo el ancho
pd.set_option('display.expand_frame_repr', False)

# Opcional: Aumenta el número máximo de columnas a mostrar (por ejemplo, 50)
pd.set_option('display.max_columns', 100)

# Opcional: Aumenta el ancho máximo de cada columna para que no se corte el texto
pd.set_option('display.max_colwidth', None)
```


```python
# Fijar semilla para reproducibilidad si se generan datos sintéticos
np.random.seed(42)

raw_data = {
    "cliente_id": [101, 102, 103, 104, 105, 105, 106, 107, 108, 109],
    "nombre": [
        "  Ana Pérez ",
        "Carlos ",
        "BEATRIZ",
        "david",
        "Eva Sanz",
        "Eva Sanz",
        "Fernando Ruiz",
        "Gema",
        "Hugo López",
        "Irene",
    ],
    "edad": [24, "31", np.nan, 45, 29, 29, 150, 38, -5, "cuarenta"],
    "fecha_alta": [
        "2023-01-15",
        "15/02/2023",
        "2023-03-01",
        "sin_fecha",
        "2023-05-10",
        "2023-05-10",
        "2023-06-20",
        "2023-07-12",
        "2023/08/01",
        "N/D",
    ],
    "ingresos_anuales": [
        28000.0,
        34000.5,
        42000.0,
        np.nan,
        31000.0,
        31000.0,
        29000.0,
        850000.0,
        np.nan,
        22000.0,
    ],
    "categoria": [
        "premium",
        " standard",
        "PREMIUM",
        "Standard",
        "basic",
        "basic",
        "desconocido",
        "premium",
        "basic",
        "?",
    ],
    "es_activo": [1, 1, 0, 1, 1, 1, 0, 1, np.nan, 0],
}

df = pd.DataFrame(raw_data)
df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cliente_id</th>
      <th>nombre</th>
      <th>edad</th>
      <th>fecha_alta</th>
      <th>ingresos_anuales</th>
      <th>categoria</th>
      <th>es_activo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>101</td>
      <td>Ana Pérez</td>
      <td>24</td>
      <td>2023-01-15</td>
      <td>28000.0</td>
      <td>premium</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>Carlos</td>
      <td>31</td>
      <td>15/02/2023</td>
      <td>34000.5</td>
      <td>standard</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>BEATRIZ</td>
      <td>NaN</td>
      <td>2023-03-01</td>
      <td>42000.0</td>
      <td>PREMIUM</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>104</td>
      <td>david</td>
      <td>45</td>
      <td>sin_fecha</td>
      <td>NaN</td>
      <td>Standard</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.0</td>
      <td>basic</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.0</td>
      <td>basic</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>106</td>
      <td>Fernando Ruiz</td>
      <td>150</td>
      <td>2023-06-20</td>
      <td>29000.0</td>
      <td>desconocido</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>107</td>
      <td>Gema</td>
      <td>38</td>
      <td>2023-07-12</td>
      <td>850000.0</td>
      <td>premium</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>108</td>
      <td>Hugo López</td>
      <td>-5</td>
      <td>2023/08/01</td>
      <td>NaN</td>
      <td>basic</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>9</th>
      <td>109</td>
      <td>Irene</td>
      <td>cuarenta</td>
      <td>N/D</td>
      <td>22000.0</td>
      <td>?</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
</div>



## 2.2. Auditoría de los datos

Antes de modificar un solo registro, es necesario realizar un análisis cuantitativo del estado del DataFrame. En proyectos con millones de filas no es posible examinar los datos visualmente, por lo que necesitamos analizar cada una de las columnas usando Python.


```python
# 1. Resumen estructural: tipos asignados y uso de memoria real
print()
print("********************** USO DE MEMORIA **********************")
df.info(memory_usage="deep")

# 2. Resumen estadístico de variables numéricas y categóricas
print()
print("********************** RESUMEN ESTADÍSTICO **********************")
print(df.describe(include="all"))

# 3. Diagnóstico exhaustivo de valores ausentes (conteo y porcentaje relativo)
print()
print("********************** NULOS **********************")
nulos_resumen = pd.DataFrame(
    {
        "conteo_nulos": df.isna().sum(),
        "porcentaje_nulos": (df.isna().mean() * 100).round(2),
    }
).sort_values(by="porcentaje_nulos", ascending=False)

print(nulos_resumen)
```

    
    ********************** USO DE MEMORIA **********************
    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 10 entries, 0 to 9
    Data columns (total 7 columns):
     #   Column            Non-Null Count  Dtype  
    ---  ------            --------------  -----  
     0   cliente_id        10 non-null     int64  
     1   nombre            10 non-null     object 
     2   edad              9 non-null      object 
     3   fecha_alta        10 non-null     object 
     4   ingresos_anuales  8 non-null      float64
     5   categoria         10 non-null     object 
     6   es_activo         9 non-null      float64
    dtypes: float64(2), int64(1), object(4)
    memory usage: 2.7 KB
    
    ********************** RESUMEN ESTADÍSTICO **********************
            cliente_id    nombre  edad  fecha_alta  ingresos_anuales categoria  es_activo
    count    10.000000        10   9.0          10          8.000000        10   9.000000
    unique         NaN         9   8.0           9               NaN         7        NaN
    top            NaN  Eva Sanz  29.0  2023-05-10               NaN     basic        NaN
    freq           NaN         2   2.0           2               NaN         3        NaN
    mean    105.000000       NaN   NaN         NaN     133375.062500       NaN   0.666667
    std       2.581989       NaN   NaN         NaN     289615.453323       NaN   0.500000
    min     101.000000       NaN   NaN         NaN      22000.000000       NaN   0.000000
    25%     103.250000       NaN   NaN         NaN      28750.000000       NaN   0.000000
    50%     105.000000       NaN   NaN         NaN      31000.000000       NaN   1.000000
    75%     106.750000       NaN   NaN         NaN      36000.375000       NaN   1.000000
    max     109.000000       NaN   NaN         NaN     850000.000000       NaN   1.000000
    
    ********************** NULOS **********************
                      conteo_nulos  porcentaje_nulos
    ingresos_anuales             2              20.0
    edad                         1              10.0
    es_activo                    1              10.0
    cliente_id                   0               0.0
    nombre                       0               0.0
    fecha_alta                   0               0.0
    categoria                    0               0.0


Algunas cosas en las que nos tenemos que fijar aquí:

- La columna `Dtype` nos mostrará los tipos inferidos al cargar los datos por lo que habrá que analizarla para buscar:
  - Campos que infiere como `object`, ocurre cuando hay al menos un valor que no encaja con el resto (por ejemplo, una cadena de texto en una columna de números)
  - Las fechas también las infiere siempre como `object`. Hay que localizar estos campos para realizar nosotros manualmente el cambio de tipo.
  - Si hay nulos en columnas de enteros, los convertirá a `float64`, ya que NumPy/Pandas un array entero no admite `np.nan`.
- En cuanto al resumen estadístico:
  - El valor `unique` muestra los valores únicos, interesante para saber qué campos se pueden convertir a `category`. Aquí no lo hemos hecho, pero lo ideal es hacerlo una vez que se hayan uniformado los valores del campo (todo a minúsculas, sin espacios, ...)
  - La media, mediana y percentiles muestra información muy relevante sobre cómo están distribuidos los datos, aunque ya veremos más adelante con más detalle su significado.
- La salida de nulos nos da una idea de cuántos nulos hay por cada campo.

## 2.3. Normalización léxica y limpieza vectorizada de cadenas

Los datos ingeridos desde sistemas heterogéneos (formularios web, bases de datos transaccionales, ficheros CSV o volcados JSON de APIs externas) presentan habitualmente anomalías de texto: espacios no imprimibles, caracteres invisibles y cadenas centinela que enmascaran la ausencia real de datos.

### El antipatrón de iteración frente al accesor vectorizado `.str`

En Python estándar, el procesamiento de cadenas suele realizarse iterando con bucles `for`, comprensiones de listas o delegando en `.apply(lambda x: ...)` de Pandas. En Big Data, este enfoque es un **antipatrón crítico de rendimiento**:

1. **Sobrecarga del intérprete:** `.apply()` ejecuta la función fila a fila dentro de la máquina virtual de Python, sufriendo la contención del GIL (*Global Interpreter Lock*) y el salto continuo de punteros en memoria (*pointer chasing*).
2. **Optimizaciones de bajo nivel:** El accesor `.str` de Pandas traslada las operaciones a extensiones compiladas en C/Cython. Procesa arrays contiguos en memoria y gestiona de forma nativa la propagación de valores nulos: si un elemento es `np.nan` o `None`, `.str` lo ignora sin lanzar excepciones `AttributeError` (algo que rompería inmediatamente un `.apply(lambda x: x.strip())`).

| Enfoque                          | Mecánica de ejecución                   | Rendimiento relativo                   | Manejo de nulos            |
| -------------------------------- | --------------------------------------- | -------------------------------------- | -------------------------- |
| `for row in df.itertuples()`     | Intérprete Python puro                  | Muy lento ($\times 100$ tiempo de CPU) | Manual (propenso a error)  |
| `df['col'].apply(lambda x: ...)` | Bucle Python bajo abstracción de Pandas | Lento ($\times 15 - \times 30$)        | Requiere `if pd.notna(x)`  |
| `df['col'].str.operacion()`      | Rutinas vectorizadas en C / Cython      | **Óptimo**                             | Automático (propaga `NaN`) |


Algunas operaciones de limpieza de texto que se realizan son:

1. **Supresión de espacios residuales y caracteres de control**: los espacios en blanco al inicio o al final de una cadena (*leading/trailing whitespaces*) son la causa principal de claves no coincidentes en operaciones `JOIN` o `drop_duplicates()`.
    - `.str.strip()`: elimina espacios en blanco, tabuladores (`\t`) y saltos de línea (`\n`, `\r`) en ambos extremos.
    - `.str.replace(r'\s+', ' ', regex=True)`: colapsa múltiples espacios internos consecutivos en un único espacio estándar.

2. **Homogeneización de capitalización**: para columnas que representan entidades o categorías, se debe adoptar un formato canónico:
    - `.str.lower()`: recomendado para variables categóricas (`"PREMIUM"` a `"premium"`), facilitando agrupaciones y filtros.
    - `.str.title()` / `.str.capitalize()`: adecuado para nombres propios y entidades legibles (`"david"` a `"David"`).

3. **Neutralización de valores centinela**: muchos sistemas heredados rellenan campos ausentes con literales de texto en lugar de nulos reales: `"?"`, `"desconocido"`, `"n/d"`, `"none"`, `"null"` o cadenas vacías `""`. Para Pandas, `"?"` es un texto válido con longitud 1; no lo cuantifica como `NaN` en `isna()`, distorsionando la auditoría de nulos. Estos valores deben identificarse y mapearse formalmente a `np.nan` mediante `.replace()`.

4. **Reducción de consumo de RAM con `category`**: una columna de texto con valores repetidos almacenada como `object` guarda en cada celda un puntero de 64 bits hacia un objeto `PyObject` independiente en el *heap*, disparando la huella de memoria. Al aplicar `.astype("category")`, Pandas implementa internamente una **codificación por diccionario**:
    - Extrae los valores únicos y los indexa numéricamente en una tabla interna de categorías.
    - Almacena la columna principal como un array compacto de enteros de 8 o 16 bits (`int8` o `int16`), reduciendo el consumo de RAM entre un 80% y un 90%.


```python
# 1. Normalización de cadenas de texto libre (Nombres propios)
# Eliminación de espacios perimetrales y conversión a formato título (Title Case)
df["nombre"] = df["nombre"].str.strip().str.title()

# 2. Homogeneización de variables categóricas textuales
# Pasamos a minúsculas y eliminamos espacios parásitos
df["categoria"] = df["categoria"].astype(str).str.strip().str.lower()

# 3. Detección y sustitución de valores centinela por NaN formal
# Catálogo de cadenas que representan ausencia de dato en los sistemas de origen
valores_centinela = ["?", "desconocido", "n/d", "none", "nan", "sin_fecha", ""]

# Sustitución vectorizada en la columna categórica
df["categoria"] = df["categoria"].replace(valores_centinela, np.nan)

# 4. Optimización de memoria: CategoricalDtype
# Tras homogeneizar ('premium' y 'PREMIUM' son ahora idénticos) y aislar nulos,
# convertimos a tipo categórico para optimizar memoria RAM.
memoria_antes = df["categoria"].memory_usage(deep=True)
df["categoria"] = df["categoria"].astype("category")
memoria_despues = df["categoria"].memory_usage(deep=True)

print(f"Consumo 'categoria' antes: {memoria_antes} bytes")
print(f"Consumo 'categoria' después: {memoria_despues} bytes")
print(f"Valores canónicos únicos:\n{df['categoria'].cat.categories.tolist()}")
```

    Consumo 'categoria' antes: 583 bytes
    Consumo 'categoria' después: 572 bytes
    Valores canónicos únicos:
    ['basic', 'no_registrado', 'premium', 'standard']


## 2.3. Integridad de claves y gestión de duplicados

La duplicidad de registros falsea la cardinalidad de los datos e invalida las métricas de evaluación en modelos predictivos. Si un registro idéntico aparece en el conjunto de entrenamiento (*train*) y en el de prueba (*test*), se produce una **fuga de datos**, reportando un rendimiento artificialmente optimista.

Existen dos tipos de duplicados:

1. **Duplicados completos:** todas las columnas contienen exactamente los mismos valores.
2. **Duplicados lógicos o de clave primaria:** registros que repiten el identificador de negocio pero difieren en algún atributo secundario (frecuente en fallos de ingesta o reintentos de red).


```python
# Identificar duplicados completos
print("Duplicados exactos:", df.duplicated().sum())

# Mostrar los registros duplicados completos para inspección previa
filas_duplicadas = df[df.duplicated(keep=False)]
print(filas_duplicadas)

# Eliminación de duplicados completos conservando la primera instancia
df = df.drop_duplicates(keep="first").reset_index(drop=True)

# Validación por clave de negocio: comprobar si hay IDs de cliente repetidos
duplicados_id = df.duplicated(subset=["cliente_id"], keep=False)
if duplicados_id.any():
    print(
        "Alerta: Se detectaron IDs repetidos con datos divergentes:\n",
        df[duplicados_id],
    )
    # Resolución: conservar el registro con información más reciente o más completa
    df = df.drop_duplicates(subset=["cliente_id"], keep="first").reset_index(
        drop=True
    )
```

    Duplicados exactos: 1
       cliente_id    nombre edad  fecha_alta  ingresos_anuales categoria  es_activo
    4         105  Eva Sanz   29  2023-05-10           31000.0     basic        1.0
    5         105  Eva Sanz   29  2023-05-10           31000.0     basic        1.0


## 2.4. Integridad de claves y gestión de duplicados

La persistencia de duplicados falsea la cardinalidad de las entidades del modelo dimensional, distorsiona los cálculos agregados (sumas de ventas, métricas medias) e invalida los modelos predictivos. Si un mismo registro aparece simultáneamente en los particionados de entrenamiento (*train*) y de validación (*test*), se produce una **fuga de datos** (*data leakage*): el modelo memoriza el patrón en lugar de generalizar, arrojando métricas de rendimiento artificialmente optimistas que colapsan al desplegar en producción.

```
                    Evaluación de duplicados (df.duplicated)
                                       │
               ┌───────────────────────┴───────────────────────┐
               ▼                                               ▼
      Duplicados completos                           Conflictos de clave
     (Todas las columnas)                         (subset=['cliente_id'])
               │                                               │
               ▼                                               ▼
     Eliminación directa                           Inspección con keep=False
      con keep='first'                             (Desvío a cuarentena)
                                                               │
                                               ┌───────────────┴───────────────┐
                                               ▼                               ▼
                                     Resolución determinista          Consolidación
                                   (Prioridad por completitud         (Coalescing de
                                      o fecha más reciente)        atributos dispersos)

```

Podemos encontrar dos casuísticas cuando buscamos duplicados en un dataframe:

1. **Duplicados completos (tuplas idénticas):** todas las columnas de la fila contienen exactamente los mismos valores. Suelen generarse por reintentos automáticos de conexión en APIs (*network retries*), ejecuciones duplicadas de tareas en orquestadores (Airflow, Dagster) o fallos de idempotencia en consumidores de colas de mensajes (Kafka, RabbitMQ). Su eliminación es directa y no destructiva.
2. **Duplicados lógicos o de clave de negocio (conflictos de entidad):** registros que comparten el mismo identificador unívoco (por ejemplo, `cliente_id`) pero difieren en atributos secundarios. Responden a condiciones de carrera en bases de datos distribuidas, actualizaciones parciales no versionadas o solapamiento de fuentes transaccionales. Estos registros no se pueden eliminar arbitrariamente, sino que primero hay que estudiarlos para tomar la decisión más adecuada en cada caso.

Para analizar los registros debemos utilizar el método `.duplicated()` que examina las filas y devuelve una máscara booleana (`True` para duplicados, `False` para únicos). Su comportamiento está gobernado por el parámetro `keep`:

| Valor del parámetro            | Comportamiento en la máscara booleana                                          | Aplicación práctica en ingeniería                                                |
| ------------------------------ | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| `keep='first'` *(por defecto)* | Marca como `True` todas las repeticiones a partir de la segunda aparición.     | Purgas secuenciales de duplicados exactos.                                       |
| `keep='last'`                  | Marca como `True` todas las repeticiones salvo la última instancia encontrada. | Priorización rápida en ingestas ordenadas cronológicamente.                      |
| `keep=False`                   | **Marca como `True` todas las filas implicadas**, incluida la original.        | **Auditoría y cuarentena:** aísla el grupo conflictivo completo para inspección. |

Para eliminar los duplicados utilizaremos la función `df.drop_duplicates()` usando `keep='first'`, pero si hay duplicados lógicos primero tendremos que ordenar el DataFrame asegurando que la fila prioritaria (la que queremos conservar) quede en primer lugar.

Por ejemplo, podríamos querer conservar la que tenga menor número de campos con valor nulo o con la fecha más reciente.

Siempre que eliminemos duplicados es importante encadenar `.reset_index(drop=True)` para que no queden índices discontinuos en el DataFrame


```python
# 1.- Detección y purga de duplicados exactos (Completos)
n_duplicados_exactos = df.duplicated(keep="first").sum()
print(f"Duplicados completos detectados: {n_duplicados_exactos}")

# Inspección de los registros exactos implicados
if n_duplicados_exactos > 0:
    filas_duplicadas_exactas = df[df.duplicated(keep=False)]
    print("Muestra de registros idénticos detectados:")
    print(filas_duplicadas_exactas)

# Eliminación conservando la primera instancia y reconstruyendo el índice
df = df.drop_duplicates(keep="first").reset_index(drop=True)


# 2.- Detección de colisiones por clave de negocio (cliente_id)
# Localizar colisiones de identificador con datos divergentes usando keep=False
mascara_conflictos_id = df.duplicated(subset=["cliente_id"], keep=False)

if mascara_conflictos_id.any():
    # Aislamiento en un DataFrame de cuarentena para auditoría
    df_cuarentena = df[mascara_conflictos_id].sort_values(by="cliente_id")
    print("\nALERTA: Registros con ID en conflicto desviados a cuarentena:")
    print(df_cuarentena)

    # 3.- Resolución determinista basada en reglas de calidad
    # Regla 1: Maximizar completitud (menor cantidad de NaNs)
    # Regla 2: Priorizar fecha más reciente si persiste el empate
    
    # Métrica de calidad: número de campos no nulos por fila
    df["_completitud"] = df.notna().sum(axis=1)

    # Ordenación jerárquica obligatoria antes de desduplicar
    df = (
        df.sort_values(
            by=["cliente_id", "_completitud", "fecha_alta"],
            ascending=[True, False, False]
        )
        .drop_duplicates(subset=["cliente_id"], keep="first")
        .drop(columns=["_completitud"])
        .reset_index(drop=True)
    )

print(f"\nDataFrame tras consolidar unicidad de claves: {len(df)} registros válidos.")

df
```

    Duplicados completos detectados: 0
    
    DataFrame tras consolidar unicidad de claves: 7 registros válidos.





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cliente_id</th>
      <th>nombre</th>
      <th>edad</th>
      <th>fecha_alta</th>
      <th>ingresos_anuales</th>
      <th>categoria</th>
      <th>es_activo</th>
      <th>ingresos_anuales_capped</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>101</td>
      <td>Ana Pérez</td>
      <td>24</td>
      <td>2023-01-15</td>
      <td>28000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>28000.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>Carlos</td>
      <td>31</td>
      <td>2023-02-15</td>
      <td>34000.50</td>
      <td>standard</td>
      <td>True</td>
      <td>34000.500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>Beatriz</td>
      <td>30</td>
      <td>2023-03-01</td>
      <td>42000.00</td>
      <td>premium</td>
      <td>False</td>
      <td>42000.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>True</td>
      <td>31000.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>106</td>
      <td>Fernando Ruiz</td>
      <td>30</td>
      <td>2023-06-20</td>
      <td>32500.25</td>
      <td>no_registrado</td>
      <td>False</td>
      <td>32500.250</td>
    </tr>
    <tr>
      <th>5</th>
      <td>107</td>
      <td>Gema</td>
      <td>38</td>
      <td>2023-07-12</td>
      <td>850000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>48500.625</td>
    </tr>
    <tr>
      <th>6</th>
      <td>108</td>
      <td>Hugo López</td>
      <td>30</td>
      <td>2023-08-01</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>&lt;NA&gt;</td>
      <td>31000.000</td>
    </tr>
  </tbody>
</table>
</div>



## 2.5. Coerción de tipos de datos y estandarización temporal

Tras normalizar textos y resolver duplicados, el DataFrame aún conserva columnas en tipos genéricos (`object`) debido a la heterogeneidad de los datos originales. Forzar el tipado correcto es indispensable para habilitar operaciones aritméticas, filtros cronológicos y reducir el consumo de memoria.

**Coerción numérica y el problema de los tipos enteros tradicionales**

En NumPy y en el Pandas tradicional, el tipo entero primitivo (`int64`) **no admite valores nulos**. Si una columna de enteros contiene un solo `NaN`, Pandas la promociona forzosamente a `float64`, alterando su representación (mostrando `24.0` en lugar de `24`).

Para solucionar esto, Pandas introduce los **tipos anulables (*Nullable Types*)**, que usan un array booleano interno como máscara de nulos:

- `Int64` (con **I** mayúscula): permite enteros conservando `NaN`/`<NA>`.
- `boolean`: permite valores lógicos (`True`, `False`) admitiendo valores ausentes sin degradar a `float64`.

El parámetro `errors='coerce'` es fundamental: convierte cualquier valor no parseable (como `"cuarenta"`) directamente en `NaN`, evitando que el proceso aborte con una excepción `ValueError`.

**Estandarización de fechas heterogéneas**

En ingestas de Big Data es habitual recibir fechas en múltiples formatos simultáneos (ISO 8601, formato europeo `DD/MM/YYYY`, barras o guiones). Tenemos que tener en cuenta:

- `pd.to_datetime(..., format='mixed')` permite que el parser evalúe dinámicamente cada registro de forma independiente.
- `errors='coerce'` transforma cualquier fecha corrupta o no recuperable en `NaT` (*Not a Time*, el equivalente a `NaN` en marcas temporales).


```python
# 1. Coerción numérica con tipos enteros anulables (Nullable Integer)
# 'errors="coerce"' convierte textos como "cuarenta" en NaN
df["edad"] = pd.to_numeric(df["edad"], errors="coerce")

# Convertimos al tipo Int64 de Pandas (admite enteros y nulos simultáneamente)
df["edad"] = df["edad"].astype("Int64")


# 2. Tipado booleano anulable
# Evita que la presencia de nulos fuerce la columna 'es_activo' a float64
df["es_activo"] = (
    pd.to_numeric(df["es_activo"], errors="coerce")
    .astype("boolean")
)


# 3. Estandarización de fechas multiformato
# 'format="mixed"' procesa formatos heterogéneos (YYYY-MM-DD y DD/MM/YYYY)
# Registros no válidos se convierten de forma segura en NaT
df["fecha_alta"] = pd.to_datetime(
    df["fecha_alta"], format="mixed", errors="coerce"
)


# 4. Verificación del estado de los tipos
print(df[["edad", "es_activo", "fecha_alta"]].dtypes)
df
```

    edad                   Int64
    es_activo            boolean
    fecha_alta    datetime64[ns]
    dtype: object





<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cliente_id</th>
      <th>nombre</th>
      <th>edad</th>
      <th>fecha_alta</th>
      <th>ingresos_anuales</th>
      <th>categoria</th>
      <th>es_activo</th>
      <th>ingresos_anuales_capped</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>101</td>
      <td>Ana Pérez</td>
      <td>24</td>
      <td>2023-01-15</td>
      <td>28000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>28000.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>Carlos</td>
      <td>31</td>
      <td>2023-02-15</td>
      <td>34000.50</td>
      <td>standard</td>
      <td>True</td>
      <td>34000.500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>Beatriz</td>
      <td>30</td>
      <td>2023-03-01</td>
      <td>42000.00</td>
      <td>premium</td>
      <td>False</td>
      <td>42000.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>True</td>
      <td>31000.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>106</td>
      <td>Fernando Ruiz</td>
      <td>30</td>
      <td>2023-06-20</td>
      <td>32500.25</td>
      <td>no_registrado</td>
      <td>False</td>
      <td>32500.250</td>
    </tr>
    <tr>
      <th>5</th>
      <td>107</td>
      <td>Gema</td>
      <td>38</td>
      <td>2023-07-12</td>
      <td>850000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>48500.625</td>
    </tr>
    <tr>
      <th>6</th>
      <td>108</td>
      <td>Hugo López</td>
      <td>30</td>
      <td>2023-08-01</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>&lt;NA&gt;</td>
      <td>31000.000</td>
    </tr>
  </tbody>
</table>
</div>



## 2.6. Reglas de validación de dominio y tratamiento de valores atípicos (*Outliers*)

Antes de imputar valores ausentes en el siguiente apartado, es imprescindible identificar y aislar las anomalías numéricas. Si calculásemos la media o la mediana para rellenar los `NaN` conservando valores absurdos como una edad de $-5$ o $150$ años, **estaríamos contaminando los estadísticos de imputación**.

En ingeniería de datos se distinguen dos tipos de valores anómalos que requieren tratamientos completamente distintos:

```
                            Valor numérico anómalo
                                      │
              ┌───────────────────────┴───────────────────────┐
              ▼                                               ▼
    Inconsistencia de dominio                      Valor extremo legítimo
   (Imposible biológico/físico)                 (Variabilidad real del negocio)
   Ej: edad = -5 o edad = 150                    Ej: ingresos = 850.000 €
              │                                               │
              ▼                                               ▼
   Sustituir por NaN formal                       Métodos estadísticos (IQR)
                                                              │
                                              ┌───────────────┴───────────────┐
                                              ▼                               ▼
                                    Acotamiento (Capping)           Transformación
                                      con serie.clip()               log(1 + x)

```

Según el 

### 1. Reglas semánticas de dominio

Son restricciones deterministas impuestas por la naturaleza física, biológica o contractual del dato. No requieren herramientas estadísticas:

- **Identificación:** una edad negativa ($-5$) o biológicamente inviable ($150$ años para un cliente activo) responde siempre a errores de sensor, volcados corruptos o fallos de validación en la capa de captura.
- **Acción:** no se debe adivinar el número ni eliminar la fila completa (destruiríamos información válida en el resto de columnas). Lo correcto es **sobrescribir el valor con `np.nan**`, delegando su resolución al módulo de imputación.


### 2. Detección estadística: Método del Rango Intercuartílico (*IQR*)

Para variables cuantitativas con variabilidad real (como `ingresos_anuales`), los valores extremos no son necesariamente erróneos: un cliente puede percibir $850.000\text{ \euro}$. Descartar sistemáticamente estas filas falsearía la distribución real del negocio e impediría entrenar modelos en casos críticos (como detección de fraude o clientes de alto valor).

El **Rango Intercuartílico (IQR)** es el estándar en datos no paramétricos porque se basa en cuantiles y es inmune a las asimetrías de la muestra (a diferencia del *Z-score*, cuya media y desviación típica se ven distorsionadas por el propio outlier):

$$\text{IQR} = Q_3 - Q_1$$

$$\text{Límite Inferior} = Q_1 - (1.5 \times \text{IQR})$$

$$\text{Límite Superior} = Q_3 + (1.5 \times \text{IQR})$$

* Registros por debajo del límite inferior o por encima del superior se clasifican formalmente como valores atípicos (*outliers* leves; con factor $3.0$ se catalogan como extremos).

---

### 3. Técnicas de mitigación para producción

| Técnica | Mecánica en Pandas | Ventaja | Inconveniente |
| --- | --- | --- | --- |
| **Recorte (*Trimming*)** | `df.drop(...)` | Elimina ruido severo. | Destruye tamaño muestral y puede introducir sesgo de selección. |
| **Acotamiento (*Capping* / *Winsorization*)** | `df['col'].clip(inf, sup)` | **Recomendada:** conserva todas las filas sin descalibrar algoritmos sensibles a escala. | Aplana la varianza en los extremos de la distribución. |
| **Transformación no lineal** | `np.log1p(df['col'])` | Comprime distribuciones asimétricas a la derecha (*right-skewed*). | Dificulta la interpretabilidad directa de la magnitud original. |

---

### Código de implementación

```python
import numpy as np
import pandas as pd

# -------------------------------------------------------------------------
# FASE 1: Reglas de validación semántica de dominio (Sanity Checks)
# -------------------------------------------------------------------------
# Localizamos valores imposibles y los convertimos en NaN formal
# para que sean tratados de forma homogénea en la fase de imputación.
filtro_edad_invalida = (df["edad"] < 0) | (df["edad"] > 110)

print(f"Edades biológicamente inválidas detectadas: {filtro_edad_invalida.sum()}")
df.loc[filtro_edad_invalida, "edad"] = np.nan


# -------------------------------------------------------------------------
# FASE 2: Detección estadística mediante IQR (ingresos_anuales)
# -------------------------------------------------------------------------
def calcular_umbrales_iqr(serie: pd.Series, factor: float = 1.5):
    """Calcula los límites estadísticos no paramétricos de una distribución."""
    q1 = serie.quantile(0.25)
    q3 = serie.quantile(0.75)
    iqr = q3 - q1
    limite_inf = q1 - (factor * iqr)
    limite_sup = q3 + (factor * iqr)
    return limite_inf, limite_sup


lim_inf, lim_sup = calcular_umbrales_iqr(df["ingresos_anuales"])
print(f"Umbrales IQR para ingresos: [{lim_inf:.2f} €, {lim_sup:.2f} €]")

# Auditoría de registros que superan los umbrales
mascara_outliers = (df["ingresos_anuales"] < lim_inf) | (df["ingresos_anuales"] > lim_sup)
print("Registros anómalos detectados:")
print(df.loc[mascara_outliers, ["cliente_id", "nombre", "ingresos_anuales"]])


# -------------------------------------------------------------------------
# FASE 3: Mitigación mediante Capping / Winsorization
# -------------------------------------------------------------------------
# Con .clip() restringimos los valores extremos al techo y suelo estadísticos.
# Los valores dentro del rango no sufren alteración alguna.
df["ingresos_anuales_capped"] = df["ingresos_anuales"].clip(
    lower=lim_inf, upper=lim_sup
)

print("\nComparativa tras el acotamiento (Capping):")
print(df[["cliente_id", "ingresos_anuales", "ingresos_anuales_capped"]])

```

Una vez aisladas las anomalías biológicas en `edad` y neutralizada la varianza desmedida en `ingresos_anuales`, el conjunto de datos está listo para la imputación matemática de nulos.







## 2.7. Tratamiento e imputación de valores ausentes

Habiendo forzado los tipos correctos (apartado 2.5) y neutralizado los errores de dominio como `np.nan` (apartado 2.6), el DataFrame concentra todos los valores ausentes bajo un estándar formal unificado: `np.nan` para flotantes, `<NA>` para enteros anulables y `NaT` para fechas.

La imputación a ciegas (como rellenar todas las columnas con la media) es una de las causas principales de sesgo en modelos predictivos. El método de resolución debe seleccionarse en función de la **naturaleza estadística de la pérdida** y el rol de cada variable en el sistema.

---

### Mecanismos de pérdida de datos (Taxonomía de Rubin)

Donald Rubin definió en 1976 los tres mecanismos formales que explican la ausencia de datos:

| Mecanismo | Definición formal | Ejemplo en nuestro dataset | Estrategia de ingeniería |
| --- | --- | --- | --- |
| **MCAR** (*Missing Completely at Random*) | La probabilidad de ausencia es independiente de los datos observados y de los no observados. | Un fallo de red transitorio descarta la lectura de un sensor o un usuario recarga la página a mitad del formulario. | **Descarte seguro:** Si el volumen de nulos es bajo (< 3-5%), eliminar filas (`dropna`) reduce la muestra pero **no introduce sesgo**. |
| **MAR** (*Missing at Random*) | La probabilidad de ausencia depende de otras variables observadas, pero no del propio valor faltante. | La probabilidad de que no consten los `ingresos_anuales` es mayor en clientes jóvenes (`edad < 25`), pero no del importe exacto de su sueldo. | **Imputación condicionada:** Reconstruir el valor mediante modelos o agregados por grupo (`groupby` por categoría o perfil sociodemográfico). |
| **MNAR** (*Missing Not at Random*) | La probabilidad de ausencia depende del propio valor que falta. | Clientes con ingresos extremadamente altos o muy bajos omiten deliberadamente declarar sus `ingresos_anuales` por privacidad. | **Preservar la señal:** Rellenar con la media distorsiona la distribución real. Se debe crear una columna indicadora booleana (`ingresos_anuales_isna`) o asignar una categoría explícita. |

---

### Árbol de decisión para producción

```
                             ¿Cómo resolver los valores ausentes?
                                               │
                   ┌───────────────────────────┴───────────────────────────┐
                   ▼                                                       ▼
        ¿Es clave primaria / fecha                               ¿Es una característica
         de negocio imprescindible?                              predictiva o analítica?
                   │                                                       │
                   ▼                                                       ▼
          Eliminar fila (.dropna)                                 ¿Qué porcentaje es nulo?
                                                      ┌────────────────────┴────────────────────┐
                                                      ▼                                         ▼
                                                   > 40-50%                                  < 40%
                                                      │                                         │
                                                      ▼                                         ▼
                                              Descartar columna                     ¿Qué tipo de dato es?
                                             (salvo señal MNAR)            ┌────────────────────┴────────────────────┐
                                                                           ▼                                         ▼
                                                                        Numérica                                 Categórica
                                                                           │                                         │
                                                      ┌────────────────────┴────────────────────┐                    ▼
                                                      ▼                                         ▼             Crear categoría
                                                 ¿Simétrica?                           ¿Asimétrica/Outliers?   'no_registrado'
                                                      │                                         │             o usar la moda
                                                      ▼                                         ▼
                                                Imputar Media                            Imputar Mediana
                                                                                      (global o por grupos)

```

---

### Técnicas de resolución e implementación

#### 1. Purgado de registros no negociables

Si una fila carece de su clave primaria (`cliente_id`) o del ancla temporal (`fecha_alta`), el registro pierde su validez transaccional y analítica. Intentar inventar o imputar estos valores corrompe la trazabilidad.

#### 2. Imputación univariante con estadísticos robustos

* **Media:** Válida exclusivamente en distribuciones normales (simétricas) y sin valores extremos.
* **Mediana:** Al depender de la posición y no de la magnitud, es inmune a colas pesadas y valores atípicos. Es la opción prioritaria para variables como `edad`.

#### 3. Imputación condicionada por grupos (*Group-based Imputation*)

Asignar la mediana global a `ingresos_anuales` ignora que un cliente de segmento `premium` percibe salarios sistemáticamente superiores a los del segmento `basic`. Mediante `.groupby().transform()`, se calcula e inyecta la mediana correspondiente a su segmento específico.

#### 4. Modelado explícito de la ausencia en variables categóricas

En Machine Learning, que un cliente no declare su estado o categoría suele ser un predictor de riesgo o abandono (*churn*). Reemplazar los nulos por la moda destruye ese patrón. La mejor práctica consiste en añadir una categoría formal (por ejemplo, `'no_registrado'`).

---

### Código de implementación

```python
import numpy as np
import pandas as pd

# -------------------------------------------------------------------------
# REGLA 1: Descarte de registros con identificadores o fechas irrecuperables
# -------------------------------------------------------------------------
# 'subset' restringe la purga exclusivamente a las columnas críticas de negocio
df = df.dropna(subset=["cliente_id", "fecha_alta"]).reset_index(drop=True)


# -------------------------------------------------------------------------
# REGLA 2: Imputación univariante con estadístico robusto (edad)
# -------------------------------------------------------------------------
# Calculamos la mediana sobre las edades válidas (habiendo filtrado previamente -5 y 150)
mediana_edad = df["edad"].median()
df["edad"] = df["edad"].fillna(mediana_edad).astype("Int64")


# -------------------------------------------------------------------------
# REGLA 3: Imputación condicionada por grupos (ingresos_anuales)
# -------------------------------------------------------------------------
# 3.1. Crear una bandera booleana para preservar la señal de ausencia (MNAR)
df["ingresos_anuales_is_na"] = df["ingresos_anuales"].isna().astype("int8")

# 3.2. Imputar basándose en la mediana de su categoría de cliente
df["ingresos_anuales"] = df.groupby("categoria", observed=False)[
    "ingresos_anuales"
].transform(lambda grupo: grupo.fillna(grupo.median()))

# 3.3. Fallback: si una categoría completa estuviera compuesta únicamente de NaNs,
# la mediana del grupo devolvería NaN; resolvemos estos residuos con la mediana global
df["ingresos_anuales"] = df["ingresos_anuales"].fillna(
    df["ingresos_anuales"].median()
)


# -------------------------------------------------------------------------
# REGLA 4: Imputación de categóricas mediante clase explícita
# -------------------------------------------------------------------------
# En tipos 'category', primero se debe registrar el nuevo nivel en el diccionario
# antes de poder asignar el valor, evitando un ValueError.
if "no_registrado" not in df["categoria"].cat.categories:
    df["categoria"] = df["categoria"].cat.add_categories(["no_registrado"])

df["categoria"] = df["categoria"].fillna("no_registrado")


# -------------------------------------------------------------------------
# REGLA 5: Imputación de variables lógicas/booleanas (es_activo)
# -------------------------------------------------------------------------
# En variables binarias donde el dato ausente suele ser omisión de registro,
# se asume conservadoramente el estado neutro o falso (0 / False).
df["es_activo"] = df["es_activo"].fillna(False)


# -------------------------------------------------------------------------
# 5. Auditoría de cierre de nulos
# -------------------------------------------------------------------------
print("Conteo final de nulos por columna:")
print(df.isna().sum())

```

Con todas las columnas limpias, estandarizadas y matemáticamente consistentes, el último paso consiste en encapsular estas transformaciones bajo una arquitectura declarativa lista para entornos de producción.





```python
# Regla 1: Descartar registros donde el identificador o la fecha de alta sean irrecuperables
df = df.dropna(subset=["cliente_id", "fecha_alta"]).reset_index(drop=True)

# Regla 2: Imputación univariante con estadísticos robustos
# Para 'edad', la mediana es más adecuada que la media porque no se distorsiona con valores extremos.
mediana_edad = df["edad"].median()
df["edad"] = df["edad"].fillna(mediana_edad)

# Regla 3: Imputación condicionada por grupos (Group-based Imputation)
# Es más preciso imputar ingresos basándose en la mediana de su categoría de cliente:
df["ingresos_anuales"] = df.groupby("categoria", observed=False)[
    "ingresos_anuales"
].transform(lambda grupo: grupo.fillna(grupo.median()))

# Si queda algún valor residual huérfano (por categorías donde todos los valores eran NaN):
df["ingresos_anuales"] = df["ingresos_anuales"].fillna(
    df["ingresos_anuales"].median()
)

# Regla 4: Imputación categórica mediante creación de clase explícita
# En modelos de Machine Learning, la ausencia de un dato categórico suele tener valor predictivo propio.
df["categoria"] = (
    df["categoria"]
    .cat.add_categories(["no_registrado"])
    .fillna("no_registrado")
)

df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cliente_id</th>
      <th>nombre</th>
      <th>edad</th>
      <th>fecha_alta</th>
      <th>ingresos_anuales</th>
      <th>categoria</th>
      <th>es_activo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>101</td>
      <td>Ana Pérez</td>
      <td>24</td>
      <td>2023-01-15</td>
      <td>28000.00</td>
      <td>premium</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>Carlos</td>
      <td>31</td>
      <td>2023-02-15</td>
      <td>34000.50</td>
      <td>standard</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>Beatriz</td>
      <td>30</td>
      <td>2023-03-01</td>
      <td>42000.00</td>
      <td>premium</td>
      <td>False</td>
    </tr>
    <tr>
      <th>3</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>106</td>
      <td>Fernando Ruiz</td>
      <td>150</td>
      <td>2023-06-20</td>
      <td>32500.25</td>
      <td>no_registrado</td>
      <td>False</td>
    </tr>
    <tr>
      <th>5</th>
      <td>107</td>
      <td>Gema</td>
      <td>38</td>
      <td>2023-07-12</td>
      <td>850000.00</td>
      <td>premium</td>
      <td>True</td>
    </tr>
    <tr>
      <th>6</th>
      <td>108</td>
      <td>Hugo López</td>
      <td>-5</td>
      <td>2023-08-01</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>&lt;NA&gt;</td>
    </tr>
  </tbody>
</table>
</div>



## 2.7. Detección y tratamiento de valores atípicos (*Outliers*)

Un valor atípico puede responder a un error de captura (edad = -5 o 150) o a una variabilidad extrema real (ingresos = 850.000 €). No todos los valores extremos deben eliminarse: descartar anomalías legítimas empobrece la capacidad del modelo para generalizar o para tareas de detección de fraude.

### Método del Rango Intercuartílico (*IQR - Interquartile Range*)

Es el método no paramétrico estándar, inmune a distribuciones no normales.

$$\text{IQR} = Q_3 - Q_1$$

$$\text{Límite Inferior} = Q_1 - 1.5 \times \text{IQR}$$

$$\text{Límite Superior} = Q_3 + 1.5 \times \text{IQR}$$


```python
def calcular_limites_iqr(serie: pd.Series, factor: float = 1.5):
    """Calcula los umbrales de corte para detección de outliers por IQR."""
    q1 = serie.quantile(0.25)
    q3 = serie.quantile(0.75)
    iqr = q3 - q1
    limite_inferior = q1 - (factor * iqr)
    limite_superior = q3 + (factor * iqr)
    return limite_inferior, limite_superior


# 1. Reglas de validación semántica de dominio (Sanity Checks)
# Antes de la estadística, se aplican límites físicos o biológicos obvios:
df.loc[df["edad"] < 0, "edad"] = np.nan
df.loc[df["edad"] > 110, "edad"] = np.nan
# Re-imputar valores corregidos
df["edad"] = df["edad"].fillna(df["edad"].median())

# 2. Detección estadística sobre ingresos
lim_inf_ingresos, lim_sup_ingresos = calcular_limites_iqr(
    df["ingresos_anuales"]
)
print(f"Límites IQR para ingresos: [{lim_inf_ingresos:.2f}, {lim_sup_ingresos:.2f}]")

# Identificar registros fuera de límites
outliers_ingresos = df[
    (df["ingresos_anuales"] < lim_inf_ingresos)
    | (df["ingresos_anuales"] > lim_sup_ingresos)
]
print("Registros anómalos detectados:\n", outliers_ingresos)
```

    Límites IQR para ingresos: [20499.62, 48500.62]
    Registros anómalos detectados:
        cliente_id nombre  edad fecha_alta  ingresos_anuales categoria  es_activo
    5         107   Gema    38 2023-07-12          850000.0   premium       True


Hay diferentes técnicas que podemos utilizar para tratar los valores atípicos:

1. **Recorte:** eliminar la fila. Útil si la muestra es masiva y el dato es claramente defectuoso.
2. **Acotamiento:** reemplazar los valores más allá de los umbrales por el valor del umbral superior o inferior. Mantiene el tamaño muestral sin distorsionar la varianza.
3. **Transformación logarítmica:** para distribuciones con asimetría positiva, comprimir la escala mediante $\log(1 + x)$.


```python
# Opción: Capping o acotado con .clip()
df["ingresos_anuales_capped"] = df["ingresos_anuales"].clip(
    lower=lim_inf_ingresos, upper=lim_sup_ingresos
)

df
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>cliente_id</th>
      <th>nombre</th>
      <th>edad</th>
      <th>fecha_alta</th>
      <th>ingresos_anuales</th>
      <th>categoria</th>
      <th>es_activo</th>
      <th>ingresos_anuales_capped</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>101</td>
      <td>Ana Pérez</td>
      <td>24</td>
      <td>2023-01-15</td>
      <td>28000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>28000.000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>102</td>
      <td>Carlos</td>
      <td>31</td>
      <td>2023-02-15</td>
      <td>34000.50</td>
      <td>standard</td>
      <td>True</td>
      <td>34000.500</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103</td>
      <td>Beatriz</td>
      <td>30</td>
      <td>2023-03-01</td>
      <td>42000.00</td>
      <td>premium</td>
      <td>False</td>
      <td>42000.000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>105</td>
      <td>Eva Sanz</td>
      <td>29</td>
      <td>2023-05-10</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>True</td>
      <td>31000.000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>106</td>
      <td>Fernando Ruiz</td>
      <td>30</td>
      <td>2023-06-20</td>
      <td>32500.25</td>
      <td>no_registrado</td>
      <td>False</td>
      <td>32500.250</td>
    </tr>
    <tr>
      <th>5</th>
      <td>107</td>
      <td>Gema</td>
      <td>38</td>
      <td>2023-07-12</td>
      <td>850000.00</td>
      <td>premium</td>
      <td>True</td>
      <td>48500.625</td>
    </tr>
    <tr>
      <th>6</th>
      <td>108</td>
      <td>Hugo López</td>
      <td>30</td>
      <td>2023-08-01</td>
      <td>31000.00</td>
      <td>basic</td>
      <td>&lt;NA&gt;</td>
      <td>31000.000</td>
    </tr>
  </tbody>
</table>
</div>



## 2.8. Arquitectura funcional de producción: *Method Chaining* y `.pipe()`

En la industria del software y la ingeniería de datos, el código disperso en celdas de Jupyter Notebook con mutaciones de estado sobre la misma variable (`df['x'] = ...`) es fuente recurrente de fallos silenciosos y dificulta las pruebas unitarias.

El patrón recomendado consiste en componer funciones puras mediante el método `.pipe()`, permitiendo una lectura secuencial idéntica a un pipeline declarativo de datos:


```python
def auditar_claves(data: pd.DataFrame, id_col):
    """Elimina duplicados garantizando unicidad de identificador."""
    return data.drop_duplicates(subset=[id_col], keep="first").copy()


def estandarizar_cadenas(data):
    """Homogeneiza formatos de texto y capitalización."""
    data = data.copy()
    data["nombre"] = data["nombre"].str.strip().str.title()
    data["categoria"] = (
        data["categoria"]
            .astype(str)
            .str.strip()
            .str.lower()
            .replace(["?", "desconocido", "n/d", "none", "nan"], np.nan)
    )
    return data


def transformar_tipos(data):
    """Fuerza tipado correcto con coerción de errores."""
    data = data.copy()
    data["edad"] = pd.to_numeric(data["edad"], errors="coerce")
    data["fecha_alta"] = pd.to_datetime(
        data["fecha_alta"], format="mixed", errors="coerce"
    )
    data["es_activo"] = pd.to_numeric(
        data["es_activo"], errors="coerce"
    ).astype("boolean")
    return data


def tratar_valores_extremos(data):
    """Aplica filtros de dominio y capping a distribuciones numéricas."""
    data = data.copy()
    # Corrección biológica
    data.loc[(data["edad"] < 0) | (data["edad"] > 110), "edad"] = np.nan

    # Winsorization sobre ingresos
    q1 = data["ingresos_anuales"].quantile(0.25)
    q3 = data["ingresos_anuales"].quantile(0.75)
    iqr = q3 - q1
    sup = q3 + (1.5 * iqr)
    inf = q1 - (1.5 * iqr)
    data["ingresos_anuales"] = data["ingresos_anuales"].clip(
        lower=inf, upper=sup
    )
    return data


def imputar_nulos(data):
    """Aplica estrategias de imputación univariante y por contexto."""
    data = data.copy()
    # Descarte de nulos críticos
    data = data.dropna(subset=["fecha_alta"])

    # Imputación de numéricas
    data["edad"] = data["edad"].fillna(data["edad"].median()).astype("Int64")
    data["ingresos_anuales"] = data["ingresos_anuales"].fillna(
        data["ingresos_anuales"].median()
    )

    # Imputación de categóricas
    data["categoria"] = data["categoria"].fillna("no_registrado")
    data["categoria"] = data["categoria"].astype("category")
    return data


# Pipeline completo estructurado de forma legible, reproducible y testeable
df_produccion = (
    pd.DataFrame(raw_data)
    .pipe(auditar_claves, id_col="cliente_id")
    .pipe(estandarizar_cadenas)
    .pipe(transformar_tipos)
    .pipe(tratar_valores_extremos)
    .pipe(imputar_nulos)
    .reset_index(drop=True)
)

print(df_produccion.info())
print(df_produccion)
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 7 entries, 0 to 6
    Data columns (total 7 columns):
     #   Column            Non-Null Count  Dtype         
    ---  ------            --------------  -----         
     0   cliente_id        7 non-null      int64         
     1   nombre            7 non-null      object        
     2   edad              7 non-null      Int64         
     3   fecha_alta        7 non-null      datetime64[ns]
     4   ingresos_anuales  7 non-null      float64       
     5   categoria         7 non-null      category      
     6   es_activo         6 non-null      boolean       
    dtypes: Int64(1), boolean(1), category(1), datetime64[ns](1), float64(1), int64(1), object(1)
    memory usage: 644.0+ bytes
    None
       cliente_id         nombre  edad fecha_alta  ingresos_anuales      categoria  es_activo
    0         101      Ana Pérez    24 2023-01-15         28000.000        premium       True
    1         102         Carlos    31 2023-02-15         34000.500       standard       True
    2         103        Beatriz    30 2023-03-01         42000.000        premium      False
    3         105       Eva Sanz    29 2023-05-10         31000.000          basic       True
    4         106  Fernando Ruiz    30 2023-06-20         29000.000  no_registrado      False
    5         107           Gema    38 2023-07-12         52250.625        premium       True
    6         108     Hugo López    30 2023-08-01         32500.250          basic       <NA>



```python

```
