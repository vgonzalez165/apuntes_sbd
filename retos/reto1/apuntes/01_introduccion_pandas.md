# 1.- Introducción a Pandas

Pandas no es una simple biblioteca de manipulación de tablas; es una capa de abstracción construida sobre **NumPy** diseñada para el procesamiento vectorizado y el análisis exploratorio de datos. Para trabajar de forma eficiente en entornos de Inteligencia Artificial y Big Data, el primer paso es abandonar la programación procedimental basada en bucles (`for`, `while`) y adoptar el **paradigma vectorizado y declarativo**.

## 1. El modelo mental: de la iteración a la vectorización

En Python estándar, transformar una lista implica recorrer elemento a elemento. En Pandas, las operaciones se ejecutan sobre matrices contiguas en memoria escritas en C. Esto quiere decir que, cuando yo quiero realizar una función sobre todos los elementos de una lista, en Python tradicional tengo que iterar sobre cada elemento de la lista y aplicar sobre dicho elemento la operación deseada. En cambio, en Pandas realizo la operación directamente sobre la lista (en realidad, la estructura de datos de Pandas que equivale a una lista es la serie) y ya se encarga Pandas internamente de aplicar la operación sobre cada elemento de una forma mucho más rápida y eficiente.

En el siguiente código puedes ver un ejemplo de cómo calcular los cuadrados de todos los números de una lista en Python y usando Pandas.


```python
import pandas as pd   # Para usar la librería Pandas primero debemos importarla, por costumbre se renombra como pd

N = 1_000_000   # Recuerda que en Python se puede usar el símbolo _ como separador de miles para aumentar la legibilidad

# Creamos las dos estructuras que contienen los números del 1 al 1000000
datos_list = list(range(N))
serie_pd = pd.Series(range(N))
```


```python
# Cálculo de los cuadrados en Python
%timeit [x**2 for x in datos_list]
```

    31.8 ms ± 513 µs per loop (mean ± std. dev. of 7 runs, 10 loops each)



```python
# Cálculo de los cuadrados en Pandas
%timeit serie_pd ** 2
```

    982 µs ± 67.3 µs per loop (mean ± std. dev. of 7 runs, 1,000 loops each)


En los ejemplos anteriores he usado `%timeit`, un **comando mágico** de Jupyter. Los comandos mágicos permiten realizar operaciones fuera de Python en un cuaderno de Jupyter, por ejemplo, cargar un fichero en una celda o mostrar el directorio de trabajo actual. Si ejecutas `%magic` en una celda obtendrás la ayuda completa de todos estos comandos, mientras que `%quickref` mostrará una referencia rápida de todos estos comandos.

En concreto, el comando `%timeit` sirve para medir el tiempo que tarda en ejecutarse un código determinado. Para ello, lo ejecuta múltiples veces y nos devuelve la media de estas mediciones.

Como podemos ver en la salida del comando anterior, el cálculo en Python tarda unos 32 milisegundos, mientras que la misma operación en Pandas se puede realizar en apenas 800 microsegundos, apenas un 2.5% del tiempo que llevó en Python

## 2. Las dos estructuras de datos fundamentales de Pandas

En Pandas se trabaja con dos tipos de estructuras de datos:

- **Series**: 
- **DataFrames**: una tabla compuesta por un diccionario ordenado de series que comparten un mismo índice de filas

### Series

Una serie es una columna unidimensional con tipado homogéneo y un índice explícito asociado a cada elemento


```python
edades = pd.Series([23, 45, 31, 19], index=['usr_1', 'usr_2', 'usr_3', 'usr_4'], name='edad')

edades
```




    usr_1    23
    usr_2    45
    usr_3    31
    usr_4    19
    Name: edad, dtype: int64



En el ejemplo anterior hemos creado una serie con 4 elementos cuyos índices son `usr_1`, `usr_2`, ... Si no hubiéramos indicado estos índices explícitamente Pandas hubiera asignado números secuencialmente. Observa también que Pandas ha inferido el tipo de datos y le ha asignado `int64`.

### DataFrames

Un DataFrame es una tabla compuesta por un diccionario ordenado de series que comparte un mismo índice de filas.


```python
datos = {
    'edad': [23, 45, 31, 19],
    'compras': [120.5, 340.0, 50.2, 890.1],
    'segmento': ['A', 'B', 'B', 'A']
}

df = pd.DataFrame(datos, index=['usr_1', 'usr_2', 'usr_3', 'usr_4'])
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
      <th>edad</th>
      <th>compras</th>
      <th>segmento</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_1</th>
      <td>23</td>
      <td>120.5</td>
      <td>A</td>
    </tr>
    <tr>
      <th>usr_2</th>
      <td>45</td>
      <td>340.0</td>
      <td>B</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>31</td>
      <td>50.2</td>
      <td>B</td>
    </tr>
    <tr>
      <th>usr_4</th>
      <td>19</td>
      <td>890.1</td>
      <td>A</td>
    </tr>
  </tbody>
</table>
</div>



Algunas funciones y atributos básicos de los DataFrames son:

- `df.shape`: muestra el número de filas y columnas del DataFrame
- `df.dtypes`: tipos de datos por columna
- `df.info()`: resumen de memoria y nulos
- `df.describe()`: métricas estadísticas básicas


```python
df.shape
```




    (4, 3)




```python
df.dtypes
```




    edad          int64
    compras     float64
    segmento     object
    dtype: object




```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Index: 4 entries, usr_1 to usr_4
    Data columns (total 3 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   edad      4 non-null      int64  
     1   compras   4 non-null      float64
     2   segmento  4 non-null      object 
    dtypes: float64(1), int64(1), object(1)
    memory usage: 128.0+ bytes



```python
df.describe()
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
      <th>edad</th>
      <th>compras</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>4.00000</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>29.50000</td>
      <td>350.200000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>11.47461</td>
      <td>380.507617</td>
    </tr>
    <tr>
      <th>min</th>
      <td>19.00000</td>
      <td>50.200000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>22.00000</td>
      <td>102.925000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>27.00000</td>
      <td>230.250000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>34.50000</td>
      <td>477.525000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>45.00000</td>
      <td>890.100000</td>
    </tr>
  </tbody>
</table>
</div>



## 3. Acceso, selección y el problema de las copias

El acceso a datos en Pandas suele ser la primera fuente de errores silenciosos y degradación de rendimiento. Comprender los mecanismos internos de indexación y cómo interactúan con la memoria evita comportamientos no deterministas en producción.

## 3.1. Indexación estándar (`[]`) frente a indexadores explícitos

En Python puro, los corchetes `[]` operan siempre sobre la misma dimensión. En un DataFrame, que es una estructura bidimensional, los corchetes pueden ser en ocasiones ambiguos o llevar a errores. Observa en los siguientes ejemplos cómo cambia el resultado según la forma de ponerlos.


```python
df = pd.DataFrame({
    'edad': [25, 42, 37, 19],
    'compras': [120.0, 540.5, 210.2, 85.0],
    'ciudad': ['León', 'Oviedo', 'León', 'Burgos']
}, index=['usr_1', 'usr_2', 'usr_3', 'usr_4'])

# ACCESO DIRECTO df[]: Comportamiento heterogéneo
df['edad']             # Selecciona una COLUMNA (devuelve una Series)
df[['edad', 'ciudad']] # Selecciona varias COLUMNAS (devuelve un DataFrame)
df[0:2]                # Corta FILAS por posición (slice, devuelve un subconjunto de las filas del DataFrame)
df[df['edad'] > 30]    # Filtra FILAS por máscara booleana (devuelve un subconjunto de las filas, que son las que cumplen la condición)
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
      <th>edad</th>
      <th>compras</th>
      <th>ciudad</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_2</th>
      <td>42</td>
      <td>540.5</td>
      <td>Oviedo</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>37</td>
      <td>210.2</td>
      <td>León</td>
    </tr>
  </tbody>
</table>
</div>



Como puedes ver, usar corchetes puede inducir a error, por lo que deberías reservarlos los corchetes simples `df[...]` **exclusivamente** para proyectar columnas por su nombre (`df['col']` o `df[['a', 'b']]`). Para cualquier selección bidimensional o indexada, utiliza siempre `.loc` o `.iloc`.

Tanto `.loc` como `.iloc` permiten acceder a los elementos de un DataFrame utilizando corchetes, la diferencia entre ambos radica en que `.loc` está basado en las etiquetas de los índices mientras que `.iloc`se basa en la posición. Esto, además, implica un sútil cambio en el comportamiento, ya que cuando usamos `.loc` se incluye el límite superior, pero si usamos `.iloc` el límite superior no se incluye.

Observa el siguiente código:


```python
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
      <th>edad</th>
      <th>compras</th>
      <th>ciudad</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_1</th>
      <td>25</td>
      <td>120.0</td>
      <td>León</td>
    </tr>
    <tr>
      <th>usr_2</th>
      <td>42</td>
      <td>540.5</td>
      <td>Oviedo</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>37</td>
      <td>210.2</td>
      <td>León</td>
    </tr>
    <tr>
      <th>usr_4</th>
      <td>19</td>
      <td>85.0</td>
      <td>Burgos</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Filtramos las filas desde usr_1 a usr_3 y las columnas edad y compras
df.loc['usr_1':'usr_3', ['edad', 'compras']]
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
      <th>edad</th>
      <th>compras</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_1</th>
      <td>25</td>
      <td>120.0</td>
    </tr>
    <tr>
      <th>usr_2</th>
      <td>42</td>
      <td>540.5</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>37</td>
      <td>210.2</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Filtramos todas las filas (observa la sintaxis con :) y el rango de columnas 'compras' hasta 'ciudad', ambas incluidas
df.loc[:, 'compras':'ciudad']
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
      <th>compras</th>
      <th>ciudad</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_1</th>
      <td>120.0</td>
      <td>León</td>
    </tr>
    <tr>
      <th>usr_2</th>
      <td>540.5</td>
      <td>Oviedo</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>210.2</td>
      <td>León</td>
    </tr>
    <tr>
      <th>usr_4</th>
      <td>85.0</td>
      <td>Burgos</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Las dos primeras filas y las dos primeras columnas (observa que la tercera, con índice 2, es excluida)
df.iloc[0:2, 0:2]
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
      <th>edad</th>
      <th>compras</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_1</th>
      <td>25</td>
      <td>120.0</td>
    </tr>
    <tr>
      <th>usr_2</th>
      <td>42</td>
      <td>540.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Usamos un rango negativo para indicar la última fila y todas las columnas
df.iloc[-1, :]
```




    edad           19
    compras      85.0
    ciudad     Burgos
    Name: usr_4, dtype: object



### 3.2. Acceso a escalares con `.at` y `.iat`

Cuando en un pipeline necesitas extraer o mutar **un único valor escalar**, `.loc` e `.iloc` introducen una sobrecarga innecesaria porque comprueban si el argumento es un slice, una lista o una máscara booleana. En esos casos es más útil utilizar `.at` (etiqueta) y `.iat` (posición), que es entre 5 y 10 veces más rápido para lecturas individuales.


```python
# Acceso al elemento que está en la fila 'usr_1' y columna 'edad'
df.at['usr_1', 'edad']
```




    25




```python
# Acceso al elemento en la primera fila y columna
df.iat[0, 0]
```




    25



## 3.3. Filtrado booleano y máscaras lógicas

El filtrado en Pandas se basa en construir una serie de booleanos que actúa como **máscara** sobre el eje de las filas.

Observa bien la sintaxis para construir la máscara, proyectamos una única columna del DataFrame y le aplicamos el operador de comparación sobre todo el DataFrame completo. Internamente, Pandas realizará esa comparación sobre cada uno de los elementos del DataFrame y generará otro DataFrame con los resultados de estas operaciones.


```python
# Creamos la máscara
mascara = df['edad'] >= 30
print(mascara)

# Aplicación con .loc
df_adultos = df.loc[mascara, ['ciudad', 'compras']]
df_adultos
```

    usr_1    False
    usr_2     True
    usr_3     True
    usr_4    False
    Name: edad, dtype: bool





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
      <th>ciudad</th>
      <th>compras</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_2</th>
      <td>Oviedo</td>
      <td>540.5</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>León</td>
      <td>210.2</td>
    </tr>
  </tbody>
</table>
</div>




```python
df[mascara]
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
      <th>edad</th>
      <th>compras</th>
      <th>ciudad</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>usr_2</th>
      <td>42</td>
      <td>540.5</td>
      <td>Oviedo</td>
    </tr>
    <tr>
      <th>usr_3</th>
      <td>37</td>
      <td>210.2</td>
      <td>León</td>
    </tr>
  </tbody>
</table>
</div>



Además de los operadores de comparación, podemos utilizar algunos métodos disponibles en Pandas para generar la máscara:

- `df['ciudad'].isin(['León', 'Burgos'])`: comprueba pertenencia a un conjunto.
- `df['compras'].between(100.0, 300.0)`: rango continuo inclusivo ($[a, b]$).
- `df['ciudad'].isna() / notna()`: comprobación de valores nulos.

### El problema de `SettingWithCopyWarning`

Un problema muy habitual es cuando encadenamos índices de la siguiente forma: `df[df['edad'] > 30]['compras'] = 0.0`. Aunque intuitivamente puede parecer que la sentencia anterior es correcta, si intentamos ejecutarla nos lanzará el error `SettingWithCopyWarning`. La clave para entender esto está en entender qué hace Python con la memoria del ordenador detrás de cada corchete.

Cuando le pides a Pandas un subconjunto de filas o columnas de un DataFrame, pueden ocurrir dos cosas a nivel de memoria:
- Una **vista**): Pandas te da una "lupa" que apunta directamente a las posiciones de memoria del DataFrame original. Si pintas a través de la lupa, modificas el original.
- Una **copia**: Pandas reserva un bloque de memoria nuevo en la RAM y clona los datos. Si pintas sobre la copia, el original queda intacto.

El problema es que Pandas no siempre te avisa de antemano si un filtro generará una vista o una copia (depende de cómo estén distribuidos los tipos de datos en los bloques contiguos de NumPy), es algo que no controlamos.

Veamos por qué ocurre esto. Supón que ejecutamos lo siguiente:

```python
df[df['edad'] > 30]['compras'] = 0.0
```

El intérprete Python lo descompondrá de la siguiente forma:

```python
paso_1 = df[df['edad'] > 30]  # No sabemos si el paso_1 es una vista o una copia
paso_1['compras'] = 0.0       # Modificamos paso_1
```

Aquí surge el conflicto:

- Si `paso_1` resultó ser una copia, acabas de modificar un objeto temporal en la memoria RAM.
- Al terminar esa línea, `paso_1` no está asignado a ninguna variable persistente, por lo que el recolector de basura (Garbage Collector) de Python lo destruye inmediatamente.
- El DataFrame original `df` nunca se enteró de la modificación.
- Pandas detecta esta cadena de operaciones y lanza `SettingWithCopyWarning`: "Ojo, intentaste asignar un valor sobre algo que probablemente era una copia temporal y tu cambio se ha perdido".

Para modificar datos sin riesgo, hay que fusionar la condición de filas y la columna objetivo en **un único par de corchetes**:

```python
df.loc[df['edad'] > 30, 'compras'] = 0.0
```

