```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. PERSISTENCIA DOCUMENTAL, CACHÉ Y PROCESAMIENTO ETL
Apartado:                   3.- Anonimización de datos
Resultados de aprendizaje:  ?
```

# 2.- Anonimización y protección de datos

## 2.1. Introducción y fundamentos conceptuales

La **anonimización de datos** consiste en la transformación de datos personales de tal forma que no puedan utilizarse para identificar a ningún individuo, directa ni indirectamente. No debe considerarse una técnica fija o un interruptor binario, sino un **proceso continuo basado en la gestión del riesgo**, que integra técnicas estadísticas/criptográficas y salvaguardas organizativas para evitar la reidentificación.

  ![[imgs/anonimizacion.png]]


### Conceptos clave: desidentificación, seudonimización y anonimización

Cuando hablamos de anonimización de datos, hay tres conceptos que debemos distinguir:

- **Desidentificación:** consiste en la eliminación o separación de los identificadores directos más evidentes (nombres, documentos de identidad, correos electrónicos). Es el primer paso y **no equivale a anonimizar**, ya que el cruce de datos con fuentes externas de acceso público suele permitir la reidentificación inmediata.
- **Seudonimización:** sustitución de identificadores por códigos o valores ficticios (tokens, hashes, valores cifrados). Puede ser reversible (si se conserva de forma segura una tabla de correspondencia o clave de descifrado) o irreversible. Bajo normativas como el RGPD (Reglamento General de Protección de Datos), **los datos seudonimizados siguen considerándose datos de carácter personal**, ya que el individuo aún puede ser reidentificado mediante información adicional.
- **Anonimización:** es un proceso irreversible que garantiza de forma razonable que la probabilidad de reidentificar a una persona es prácticamente nula, protegiendo tanto contra la vinculación directa como contra deducciones indirectas.

Veamos un ejemplo de cada uno de estos conceptos con los siguientes datos:

|                       | Nombre y apellidos | DNI           | Edad       | CP           | Profesión            | Diagnóstico              |
| --------------------- | ------------------ | ------------- | ---------- | ------------ | -------------------- | ------------------------ |
| **Datos orginales**   | Carlos López Pérez | 09876543X     | 43         | 24008        | Profesor informática | Arritmia cardíaca        |
| **Desidentificación** | *[eliminado]*      | *[eliminado]* | 43         | 24008        | Profesor informática | Arritmia cardíaca        |
| **Seudonimización**   | `TKN-8492-AX`      | *[eliminado]* | 43         | 24008        | Profesor informática | Arritmia cardíaca        |
| **Anonimización**     | *[eliminado]*      | *[eliminado]* | 40-49 años | León (24xxx) | Educación/Docencia   | Patología cardiovascular |


**Desidentificación**: eliminaríamos los identificadores directos, pero mediante los cuasi-identificadores se podría reidentificar, por ejemplo, cruzando el censo con un directorio de institutos o una red social.
**Seudonimización**: permite conservar los datos que identifican el registro (debería haber otra tabla que relacione los códigos asignados con los datos). Sigue siendo dato personal bajo RGDP.
**Anonimización**: se eliminan los identificadores directos y, además, se aplican técnicas como generalización o supresión para que no sea posible reidentificar. Estos datos quedan fuera del ámbito de aplicación del RGPD.


## 2.2. Tipología de variables y vectores de riesgo

Para seleccionar la técnica adecuada, primero se deben categorizar los atributos del conjunto de datos y comprender los tipos de ataques o revelaciones posibles.

### 2.2.1. Clasificación de las variables

Según el tipo, podemos clasificar los datos en uno de los siguientes grupos:

1. **Identificadores directos:** son los atributos únicos que por sí mismos identifican unívocamente a una persona (nombre completo, DNI/NIE, número de pasaporte, correo corporativo, número de tarjeta de crédito).
2. **Cuasi-identificadores o identificadores indirectos:** son los atributos que de forma aislada no revelan la identidad, pero que combinados entre sí o cruzados con fuentes externas permiten singularizar a un individuo (edad, sexo, código postal, profesión, puesto de trabajo, coordenadas GPS).
3. **Atributos sensibles u objetivo:** información relevante para el análisis o negocio, pero cuya revelación genera un daño o perjuicio potencial al individuo (diagnósticos clínicos, salario, transacciones bancarias, ideología política).
4. **Variables neutras/otras:** datos operativos o descriptivos no vinculables a personas que pueden mantenerse sin transformaciones.
    
### 2.2.2. Vectores de riesgo de divulgación

- **Reidentificación / Revelación de identidad:** capacidad de asociar directamente a una persona real con un registro del conjunto de datos (por ejemplo, revirtiendo un seudónimo débil o cruzando cuasi-identificadores con el censo electoral).    
- **Revelación de atributos:** deducir el valor de un atributo sensible de un individuo a partir del conjunto de datos, incluso si no se puede aislar con exactitud su fila específica (por ejemplo, si todas las personas de 45 años del código postal 28001 tienen la misma patología médica)
- **Revelación de inferencias:** extraer conclusiones con alta probabilidad sobre una persona que ni siquiera formaba parte del dataset original, basándose en los patrones estadísticos generales revelados.
    
## 2.3. El proceso metodológico en 5 Pasos

Todo proceso anonimización debe seguir una metodología estructurada en 5 fases:

  ![[pasos_anonimizacion.png]]

1. **Conocer los datos:** analizar la distribución, cardinalidad, valores atípicos y clasificar cada columna en identificador directo, cuasi-identificador o dato sensible. Aplicar siempre el **principio de minimización** (eliminar cualquier variable superflua que no aporte valor al análisis).
2. **Desidentificar:** suprimir o seudonimizar permanentemente todos los identificadores directos.
3. **Aplicar técnicas de anonimización a cuasi-identificadores:** reducir la granularidad o introducir ruido para evitar el reensamblaje de la identidad, buscando el balance entre privacidad y utilidad.
4. **Calcular y evaluar el riesgo:** medir la vulnerabilidad del dataset mediante métricas formales ($k$-anonimato, $l$-diversidad, unicidad).
5. **Gestionar el riesgo residual y documentar:** registrar los parámetros empleados, las transformaciones y los controles para auditorías, manteniendo las tablas de equivalencia en repositorios cifrados independientes.


## 2.4. Como anonimizar datos con Python

A continuación se detallan las técnicas fundamentales ilustradas con código en `pandas` y librerías criptográficas.

```python
import pandas as pd
import numpy as np
import hashlib
import random
```

### Técnica 1: Supresión de atributos y registros

Consiste en la eliminación permanente e irreversible de columnas (identificadores directos) o de filas atípicas (outliers) que faciliten la singularización. Debe ser una eliminación real de datos, nunca un simple ocultamiento visual de columnas.

```python
# Carga de datos de ejemplo (registro de empleados)
df = pd.read_csv("2016-Report-White-House-Staff.csv")

# 1. Eliminación de identificador directo (Name) y columna sin varianza/vacía
df_suprimido = df.drop(columns=['Name', 'White House Review'])

# 2. Supresión de registros atípicos (outliers extremos en salario)
# Los salarios aislados de 0 o extremadamente altos son vectores de reidentificación
q_low = df_suprimido['Salary'].quantile(0.01)
q_high = df_suprimido['Salary'].quantile(0.99)
df_sin_outliers = df_suprimido[(df_suprimido['Salary'] >= q_low) & (df_suprimido['Salary'] <= q_high)]
```

### Técnica 2: Seudonimización (tokens, hashing y cifrado)

Sustituye identificadores directos por valores codificados. Se aplica cuando se necesita conservar la relación entre registros de un mismo individuo a lo largo del tiempo o entre diferentes tablas.
#### A. Identificador aleatorio (tokenización)

Asigna un identificador único aleatorio a cada clave.

```python
# Mapeo consistente de nombres a IDs aleatorios
nombres_unicos = df['Name'].unique()
token_map = {nombre: f"EMP_{random.randint(100000, 999999)}" for nombre in nombres_unicos}

df['ID_Token'] = df['Name'].map(token_map)
```

#### B. Hashing con salting

Las funciones hash simples (como MD5 o SHA-256 directo) son deterministas y vulnerables a ataques de diccionario o tablas arcoíris (_rainbow tables_). Para garantizar robustez, siempre debe incorporarse un valor secreto aleatorio (**Salt**):

```python
SALT = "ClaveSecreta2026!#"

def generar_hash_con_salt(texto: str) -> str:
    # Concatenar salt secreto con el identificador directo
    texto_salado = texto + SALT
    return hashlib.sha256(texto_salado.encode('utf-8')).hexdigest()

df['Hash_Seguro'] = df['Name'].apply(generar_hash_con_salt)
```

#### C. Cifrado reversible

Permite al propietario del dato recuperar la identidad original bajo circunstancias justificadas.

```python
from cryptography.fernet import Fernet

# Generación y resguardo seguro de la clave
clave_secreta = Fernet.generate_key()
cipher_suite = Fernet(clave_secreta)

# Función de cifrado de campos
def cifrar_valor(valor: str) -> str:
    return cipher_suite.encrypt(valor.encode('utf-8')).decode('utf-8')

df['Nombre_Cifrado'] = df['Name'].apply(cifrar_valor)

# Separar y guardar la tabla de correspondencia en un medio seguro
tabla_correspondencia = df[['Name', 'ID_Token', 'Nombre_Cifrado']]
tabla_correspondencia.to_csv("mapeo_identidades_restringido.csv", index=False)

# Eliminar el nombre original del dataset de trabajo
df_desidentificado = df.drop(columns=['Name'])
```

### Técnica 3: Enmascaramiento de caracteres

Reemplaza parte de la cadena por caracteres comodín (`*` o `X`). Muy utilizado en códigos de serie, identificadores semánticos, tarjetas o direcciones IP donde preservar una parte del contexto es útil para el análisis.

```python
# Ejemplo con identificadores alfanuméricos tipo "SLU005-01" o "BUE126-01"
def enmascarar_codigo(codigo: str, caracteres_visibles: int = 3) -> str:
    if pd.isna(codigo):
        return codigo
    return codigo[:caracteres_visibles] + "*" * (len(codigo) - caracteres_visibles)

# Entrada: BUE126-01 -> Salida: BUE******
df_antenas['identificador_enmascarado'] = df_antenas['identificador'].apply(enmascarar_codigo)
```

### Técnica 4: Generalización y recodificación

Reduce deliberadamente la precisión de los datos agrupando valores específicos en categorías, intervalos o escalas taxonómicas más amplias.

#### A. Redondeo numérico / Reducción de escala

```python
# Redondeo de salarios al millar más cercano (ej. 42.613 -> 43.000)
df['Salario_Redondeado'] = df['Salary'].round(-3)
```

#### B. Agrupación en rangos o intervalos (Binning)

```python
# Discretización en función de tramos salariales
bins = [0, 50000, 100000, np.inf]
etiquetas = ['Nivel_Bajo', 'Nivel_Medio', 'Nivel_Alto']
df['Tramo_Salarial'] = pd.cut(df['Salary'], bins=bins, labels=etiquetas)
```

#### C. Truncamiento de valores extremos

Sustituye valores atípicos que queden en los extremos superior e inferior por umbrales fijos para evitar su singularización:

```python
# Ningún valor se muestra por debajo de 45.000 ni por encima de 165.000
df['Salario_Truncado'] = df['Salary'].clip(lower=45000, upper=165000)
```

#### D. Generalización jerárquica geográfica

```python
# Pasar de una ubicación muy granular (Calle/Municipio) a Provincia o Región
# Se descartan las coordenadas y municipios exactos
df_antenas_generalizado = df_antenas.drop(columns=['latitud', 'longitud', 'municipio'])
# Se mantiene únicamente la provincia o comunidad autónoma
```

### Técnica 5: Perturbación de datos y adición de ruido

Modifica ligeramente los valores numéricos o temporales añadiendo ruido o distorsión controlada, conservando la distribución estadística media sin reflejar los valores individuales exactos.

```python
# Perturbación de coordenadas geográficas (GPS)
def perturbar_coordenada(valor, delta_max = 0.05, decimales = 2):
    # Añade ruido aleatorio uniforme en [-delta_max, delta_max] y redondea
    ruido = random.uniform(-delta_max, delta_max)
    return round(valor + ruido, decimales)

df_antenas['lat_perturbada'] = df_antenas['latitud'].apply(perturbar_coordenada)
df_antenas['lon_perturbada'] = df_antenas['longitud'].apply(perturbar_coordenada)
```

### Técnica 6: Intercambio de datos

Permuta los valores de una columna entre diferentes registros. Esta técnica rompe la correlación entre variables a nivel de individuo, pero mantiene intactas las distribuciones marginales globales de cada atributo.

```python
# Intercambio aleatorio de la columna "Position Title"
# Útil cuando no se requiere estudiar la relación entre salario y cargo específico
df['Cargo_Intercambiado'] = np.random.permutation(df['Position Title'].values)
```

### Técnica 7: Agregación de datos

En lugar de publicar microdatos registro a registro, se publican estadísticas resumidas (recuentos, medias, desviaciones típicas).

```python
# Agregación por departamento y categoría laboral
df_agregado = df.groupby(['Department', 'Status']).agg(
    total_empleados=('Salary', 'count'),
    salario_medio=('Salary', 'mean'),
    salario_mediana=('Salary', 'median'),
    desviacion=('Salary', 'std')
).reset_index()

# Regla de corte (K-umbral): Suprimir grupos con menos de 5 individuos
df_agregado_seguro = df_agregado[df_agregado['total_empleados'] >= 5]
```

## 2.5. Modelos formales de privacidad y métricas de riesgo

Para certificar que un conjunto de datos desidentificado es verdaderamente anónimo, se deben aplicar modelos matemáticos formales de evaluación de riesgo.

### 2.5.1. $k$-Anonimato

Un conjunto de datos cumple **$k$-anonimato** si la información de cada individuo contenida en el dataset no puede distinguirse de al menos $k - 1$ individuos cuyos cuasi-identificadores son exactamente iguales. A cada uno de estos grupos se le denomina **clase de equivalencia**.

El objetivo es impedir la reidentificación por vinculación de identidad.

```python
def evaluar_k_anonimato(dataframe, cuasi_identificadores):
    """Calcula el tamaño de la clase de equivalencia más pequeña."""
    tamanos_clases = dataframe.groupby(cuasi_identificadores).size()
    return int(tamanos_clases.min())

# Ejemplo: evaluar con los atributos de sexo, edad y provincia
k_actual = evaluar_k_anonimato(df, ['Edad_Rango', 'Sexo', 'Provincia'])
print(f"El dataset cumple {k_actual}-anonimato.")
```

Si $k=1$, significa que existe al menos un registro único en la población que puede ser reidentificado con facilidad.

### 2.5.2. $l$-Diversidad

El $k$-anonimato no protege contra la **revelación de atributos** si todos los miembros de una clase de equivalencia comparten el mismo valor sensible (ataque de homogeneidad).

Una clase de equivalencia tiene **$l$-diversidad** si contiene al menos $l$ valores "bien representados" para cada atributo sensible.

```python
def evaluar_l_diversidad(dataframe, cuasi_identificadores, atributo_sensible):
    """Calcula el número mínimo de valores distintos del atributo sensible en cualquier clase de equivalencia."""
    l_valores = dataframe.groupby(cuasi_identificadores)[atributo_sensible].nunique()
    return int(l_valores.min())

l_actual = evaluar_l_diversidad(df, ['Edad_Rango', 'Sexo', 'Provincia'], 'Diagnostico')
print(f"El dataset cumple {l_actual}-diversidad.")
```

### 2.5.3. $t$-Cercanía

La $l$-diversidad no impide ataques de inferencia cuando los valores sensibles de una clase de equivalencia tienen una distribución muy desproporcionada respecto a la población general.

Una clase de equivalencia satisface **$t$-cercanía** si la distancia estadística (por ejemplo, distancia _Earth Mover's Distance_) entre la distribución del atributo sensible en esa clase y la distribución del atributo en todo el conjunto de datos no supera un umbral $t$.
    
      
### 2.5.4. Privacidad Diferencial ($\epsilon$-Differential Privacy)

Es el estándar matemático moderno. Garantiza que la salida de una consulta o cálculo estadístico sobre una base de datos sea prácticamente indistinguible con o sin la presencia de los datos de cualquier individuo específico, inyectando un ruido calibrado (mecanismos de Laplace o Gaussiano) proporcional a la sensibilidad de la consulta y a un presupuesto de privacidad $\epsilon$.

  

## 2.6. Ejemplo de un pipeline que integra todo el flujo de trabajo

El siguiente script integra todas las fases en un flujo de trabajo reproducible:

```python
import pandas as pd
import numpy as np

def pipeline_anonimizacion(df_raw):
    # Paso 1: Conocer y desidentificar identificadores directos
    columnas_a_eliminar = ['Nombre', 'DNI', 'Email', 'Telefono']
    df_proc = df_raw.drop(columns=[col for col in columnas_a_eliminar if col in df_raw.columns])
    
    # Paso 2: Generalizar cuasi-identificadores numéricos (Edad)
    bins_edad = [0, 18, 30, 45, 65, 120]
    etiquetas_edad = ['<18', '18-29', '30-44', '45-64', '65+']
    df_proc['Edad_Generalizada'] = pd.cut(df_proc['Edad'], bins=bins_edad, labels=etiquetas_edad)
    df_proc = df_proc.drop(columns=['Edad'])
    
    # Paso 3: Generalizar cuasi-identificadores geográficos (Código Postal -> Provincia)
    # Por ejemplo, quedándonos con los 2 primeros dígitos del código postal español
    df_proc['Provincia_Cod'] = df_proc['Codigo_Postal'].astype(str).str[:2]
    df_proc = df_proc.drop(columns=['Codigo_Postal'])
    
    # Paso 4: Perturbar variables cuantitativas sensibles secundarias
    df_proc['Ingresos_Aprox'] = df_proc['Ingresos'].apply(lambda x: round(x, -3))
    df_proc = df_proc.drop(columns=['Ingresos'])
    
    # Paso 5: Filtrado de registros únicos para asegurar k-anonimato mínimo (k >= 3)
    clases_qid = ['Edad_Generalizada', 'Provincia_Cod', 'Sexo']
    conteos = df_proc.groupby(clases_qid)['Sexo'].transform('count')
    df_anonimizado = df_proc[conteos >= 3].copy()
    
    return df_anonimizado
```



## 2.7. Matriz de Decisión de Técnicas

|**Tipo de Variable**|**Técnica Primaria**|**Técnica Secundaria**|**Pérdida de Utilidad**|**Nivel de Protección**|
|---|---|---|---|---|
|**Identificadores Directos** (DNI, Nombre)|Supresión (Drop)|Seudonimización / Hashing con Salt|Nula (para fines analíticos)|Muy Alto|
|**Identificadores Alfanuméricos** (Códigos)|Enmascaramiento parcial|Tokenización determinista|Baja|Medio - Alto|
|**Edad / Fechas**|Agrupación en intervalos (Binning)|Desplazamiento de fechas (Date shifting)|Media|Alto|
|**Ubicaciones (Dirección/GPS)**|Reducción de granularidad (Provincia)|Perturbación con ruido estocástico|Media - Alta|Alto|
|**Cargos / Ocupaciones**|Generalización taxonómica|Intercambio aleatorio (_Swapping_)|Media|Medio|
|**Salarios / Transacciones**|Redondeo a millares|Top/Bottom-coding (Truncamiento)|Baja|Medio - Alto|


## 2.8. Gobernanza y Marco Legal (RGPD)

1. **Documentación del Proceso:** Es obligatorio documentar las decisiones de diseño tomadas, los parámetros elegidos (por ejemplo, el valor de $k$, el ruido añadido o los tramos de agrupación) y las justificaciones del equilibrio entre privacidad y utilidad.
2. **Custodia de Claves:** Si se utiliza seudonimización o cifrado reversible, las claves criptográficas y tablas de correspondencia deben almacenarse en repositorios independientes con control de acceso estricto y auditoría de accesos.
3. **Evaluación de Reidentificación Periódica:** Los avances en potencia de cómputo y la aparición de nuevos conjuntos de datos abiertos en internet pueden volver vulnerable un dataset anonimizado tiempo atrás. Deben realizarse pruebas periódicas de reidentificación y análisis de unicidad residual.