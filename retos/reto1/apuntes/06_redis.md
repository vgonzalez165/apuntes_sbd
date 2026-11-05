```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. PERSISTENCIA DOCUMENTAL, CACHÉ Y PROCESAMIENTO ETL
Apartado:                   6.- Redis
Resultados de aprendizaje:  ?
```

# 6. Bases de datos clave-valor: Redis

**Redis** (Remote Dictionary Server) es un almacén de estructuras de datos en memoria de tipo clave-valor. En arquitecturas de Inteligencia Artificial y Big Data se utiliza principalmente por su baja latencia (lecturas y escrituras en sub-milisegundos) para servir datos en tiempo real a modelos en producción (Feature Store), cachear predicciones costosas y gestionar colas o rankings dinámicos.

Para interactuar con Redis desde Python se utiliza la librería oficial `redis-py`.

## 6.1.- Preparación del entorno y conexión desde Python

Por defecto, Redis devuelve las respuestas en formato `bytes` (`b'valor'`). Configurar `decode_responses=True` convierte automáticamente las salidas a `str` de Python, facilitando su manipulación. 

```python
import redis

# Establecer conexión con el servidor local
r = redis.Redis(
    host='localhost',
    port=6379,
    db=0,                   
    decode_responses=True
)

# Comprobar que la conexión está activa
if r.ping():
    print("Conexión exitosa a Redis")
```

En Redis se sigue una convención de nombres para asignar nombre a las claves ya que no existen tablas con en las bases de datos SQL. Para organizar los datos y evitar colisiones, el estándar en la industria es separar niveles jerárquicos mediante dos puntos (`:`). Ejemplo: `usuario:1045:perfil` o `modelo:fraude:v2`.


## 6.2.. Tipo de dato: `String`

Es la estructura más básica de Redis. Almacena cadenas de texto, números (enteros o flotantes) o datos binarios serializados (hasta 512 MB). En cualquier arquitectura de software es la opción ideal para guardar **sesiones temporales** (como códigos de verificación por SMS), **cachear respuestas JSON** para no saturar la base de datos principal, y gestionar **contadores en tiempo real** (visitas a una página, descargas o control de *rate limiting*).

Los principales métodos en Python son:

- `set(clave, valor, ex=segundos)`: guarda un valor con un tiempo de expiración opcional (TTL) en segundos.
- `get(clave)`: recupera el valor asociado a la clave (`None` si no existe o ya expiró).
- `ttl(clave)`: devuelve los segundos restantes antes de que la clave caduque (`-1` si no tiene caducidad, `-2` si ya no existe).
- `incr(clave, cantidad)` / `decr(clave, cantidad)`: incrementa o decrementa de forma atómica un valor entero.

Veamos un ejemplo práctico de como implementar códigos temporales, contadores de visitas y una caché JSON

```python
import json

# 1. Almacenar un código OTP (One-Time Password) que caduca en 5 minutos (300 segundos)
r.set("auth:otp:usuario_88", "492810", ex=300)

codigo = r.get("auth:otp:usuario_88")
tiempo_restante = r.ttl("auth:otp:usuario_88")
print(f"Código activo: {codigo} (Expira en {tiempo_restante} segundos)")

# 2. Contador atómico de visitas a una página web
r.set("web:visitas:home", 100)
r.incr("web:visitas:home")        # Suma 1 -> 101
r.incr("web:visitas:home", 25)    # Suma 25 -> 126
r.decr("web:visitas:home", 6)     # Resta 6 -> 120
print(f"Total de visitas en la home: {r.get('web:visitas:home')}")

# 3. Guardar un objeto complejo (diccionario) serializado como JSON durante 1 hora
config_sistema = {"modo_mantenimiento": False, "max_conexiones": 500, "region": "eu-west"}
r.set("cache:config_global", json.dumps(config_sistema), ex=3600)

# Recuperar y volver a transformar en diccionario de Python
config_raw = r.get("cache:config_global")
if config_raw:
    config_dict = json.loads(config_raw)
    print(f"Región activa en caché: {config_dict['region']}")
```


## 6.3.- Tipo de dato: `Hash`

Un `Hash` almacena una colección de pares `campo: valor` dentro de una única clave. Es el equivalente directo a un `dict` de Python o a un registro/fila de una tabla SQL.

Su gran ventaja frente a guardar un JSON en un `String` es que **permite leer, modificar o eliminar atributos individuales** de un objeto (por ejemplo, el stock de un producto o el email de un usuario) sin tener que descargar, deserializar y volver a subir el objeto entero.

Los principales métodos en Python para manipular hashes son:

- `hset(clave, mapping=diccionario)`: Inserta o actualiza múltiples campos a la vez pasando un diccionario de Python.
- `hget(clave, campo)`: Obtiene el valor de un único campo.
- `hmget(clave, [campos])`: Obtiene los valores de una lista específica de campos en una sola consulta.
- `hgetall(clave)`: Devuelve todos los campos y valores del Hash como un diccionario de Python.
- `hincrby(clave, campo, entero)` / `hincrbyfloat(clave, campo, flotante)`: Suma o resta un valor numérico directamente sobre un campo del Hash.
- `hdel(clave, campo)`: Elimina un campo específico dentro del Hash.

A continuación tienes un ejemplo de cómo gestionar fichas de productos e inventario de un E-commerce:

```python
clave_producto = "producto:1050"

# 1. Crear la ficha de un producto con múltiples atributos
datos_producto = {
    "nombre": "Teclado Mecánico RGB",
    "categoria": "Periféricos",
    "precio": 89.99,
    "stock": 50
}
r.hset(clave_producto, mapping=datos_producto)

# 2. Leer un único atributo o varios específicos
nombre = r.hget(clave_producto, "nombre")
precio, stock = r.hmget(clave_producto, ["precio", "stock"])
print(f"Producto: {nombre} | Precio: {precio}€ | Stock inicial: {stock} uds.")

# 3. Modificar campos en vivo: se venden 3 unidades y aplicamos una rebaja de 5.50€
nuevo_stock = r.hincrby(clave_producto, "stock", -3)
nuevo_precio = r.hincrbyfloat(clave_producto, "precio", -5.50)
print(f"Tras la venta -> Stock actual: {nuevo_stock} uds. | Nuevo precio: {nuevo_precio:.2f}€")

# 4. Añadir un campo nuevo en caliente y recuperar toda la ficha completa
r.hset(clave_producto, "proveedor", "TechCorp")
ficha_completa = r.hgetall(clave_producto)
print(f"Ficha completa en Redis: {ficha_completa}")

```

Recuerda que los `Hashes` son estructuras planas (no admiten diccionarios ni listas anidadas dentro de sus campos). Además, al usar `decode_responses=True`, todos los valores devueltos por `hget` o `hgetall` llegan a Python como texto (`str`), por lo que si vas a hacer operaciones matemáticas en tu código deberás convertirlos previamente con `int()` o `float()`.


## 6.4.- Tipo de dato: `Sorted Set` (ZSET)

Un `Sorted Set` es una colección de elementos de texto **únicos** (llamados *miembros*) donde cada elemento lleva asociado un número en coma flotante llamado **score** (puntuación). Redis mantiene los miembros ordenados de forma automática y permanente de menor a mayor según su *score*. Si insertas un miembro que ya existía, no se duplica: simplemente se actualiza su puntuación y se reordena automáticamente.

Se utiliza para construir **tablas de clasificación en tiempo real** (leaderboards de videojuegos, artículos más vendidos), **colas de tareas con prioridad** o **índices numéricos** (filtrar ciudades por población, productos por rango de precio o eventos por marca de tiempo).

Los métodos principales son:

- `zadd(clave, {miembro: score, ...})`: añade uno o varios miembros con sus respectivas puntuaciones.
- `zrange(clave, inicio, fin, desc=True, withscores=True)`: devuelve los elementos según su posición en la lista ordenada. Usando `desc=True` invierte el orden (de mayor a menor puntuación).
- `zrangebyscore(clave, min, max, withscores=True)`: filtra y devuelve todos los miembros cuya puntuación se encuentre entre `min` y `max`.
- `zincrby(clave, incremento, miembro)`: suma (o resta, si es negativo) puntos al *score* de un miembro.
- `zrank(clave, miembro)` / `zrevrank(clave, miembro)`: devuelve la posición (índice base 0) de un miembro en el ranking ascendente (`zrank`) o descendente (`zrevrank`).
- `zscore(clave, miembro)`: consulta la puntuación exacta de un miembro concreto.

El siguiente ejemplo ilustra como gestionar la clasificación en vivo de un torneo online

```python
clave_torneo = "torneo:primavera:ranking"

# 1. Registrar las puntuaciones iniciales de los jugadores
puntuaciones = {
    "jugador_ana": 1500,
    "jugador_carlos": 2300,
    "jugador_elena": 1850,
    "jugador_david": 950,
    "jugador_lucia": 3100
}
r.zadd(clave_torneo, puntuaciones)

# 2. Sumar y restar puntos tras una partida (Ana gana 600 pts, Carlos pierde 150 pts)
r.zincrby(clave_torneo, 600, "jugador_ana")
r.zincrby(clave_torneo, -150, "jugador_carlos")

# 3. Obtener el Podio (Top 3 de jugadores con más puntos)
# En Redis los rangos son inclusivos: de la posición 0 a la 2 obtenemos 3 elementos
top_3 = r.zrange(clave_torneo, 0, 2, desc=True, withscores=True)
print("Podio del Torneo (Top 3):")
for pos, (jugador, puntos) in enumerate(top_3, start=1):
    print(f" #{pos} {jugador} -> {int(puntos)} puntos")

# 4. Consultar la posición y puntuación de un jugador específico (ej. 'jugador_ana')
posicion_ana = r.zrevrank(clave_torneo, "jugador_ana") # 0 es el primero
puntos_ana = r.zscore(clave_torneo, "jugador_ana")
print(f"\nEstado de Ana: Posición #{posicion_ana + 1} con {int(puntos_ana)} puntos.")

# 5. Filtrar jugadores en la categoría 'Oro' (aquellos que tienen entre 2000 y 3500 puntos)
categoria_oro = r.zrangebyscore(clave_torneo, min=2000, max=3500, withscores=True)
print(f"Jugadores en categoría Oro (2000 - 3500 pts): {categoria_oro}")
```



## 6.5. Buenas prácticas para procesamiento de datos: `Pipelines`

Cuando procesamos lotes de datos en Big Data (por ejemplo, cargar 10.000 *features* o actualizar cientos de predicciones desde un *DataFrame* de Pandas/Spark), hacer llamadas individuales (`r.hset(...)` dentro de un bucle `for`) es muy ineficiente debido a la latencia de red en cada petición (*Round-Trip Time*).

Los **Pipelines** permiten empaquetar múltiples comandos en memoria y enviarlos a Redis en un **único viaje de red**, mejorando el rendimiento drásticamente.

```python
# Crear un objeto pipeline
pipe = r.pipeline()

# Simulación de carga por lotes (Batch ingestion) de 3 eventos
lote_usuarios = [
    {"id": "101", "visitas": 5, "score_riesgo": 0.12},
    {"id": "102", "visitas": 12, "score_riesgo": 0.88},
    {"id": "103", "visitas": 2, "score_riesgo": 0.05},
]

for u in lote_usuarios:
    uid = u["id"]
    # Encolamos comandos de String, Hash y Sorted Set sin enviarlos aún por red
    pipe.incr("metricas:usuarios_procesados")
    pipe.hset(f"features:usuario:{uid}", mapping={"visitas": u["visitas"], "riesgo": u["score_riesgo"]})
    pipe.zadd("ranking:riesgo_fraude", {f"usuario:{uid}": u["score_riesgo"]})

# Ejecutar todos los comandos encolados de golpe en el servidor
respuestas = pipe.execute()
print(f"Comandos ejecutados en lote: {len(respuestas)}")
```

## 6.6.- Otros tipos de datos

Aunque en el proyecto actual únicamente utilizaremos los tres tipos de datos vistos anteriormente, Redis dispone de otros tipos de datos que pueden ser útiles en determinadas ocasiones. 

### 6.6.1.- Tipo de dato: `List` (Listas)

Las `Lists` en Redis son listas enlazadas (*linked lists*) de cadenas de texto ordenadas según su orden de inserción. A diferencia de los *arrays* tradicionales, insertar o extraer elementos por cualquiera de los dos extremos (izquierda/cabeza o derecha/cola) es extremadamente rápido, independientemente de si la lista tiene 10 o 10 millones de elementos.

Se utilizan principalmente para implementar **colas de trabajo** (*queues* FIFO: primero en entrar, primero en salir), **pilas** (LIFO) y **timelines o historiales de actividad reciente** (ej. "últimos 50 movimientos bancarios" o "últimos 10 artículos visitados").

Los principales métodos son:

- `lpush(clave, *valores)` / `rpush(clave, *valores)`: inserta uno o varios elementos por la izquierda (inicio) o por la derecha (final).
- `lpop(clave)` / `rpop(clave)`: extrae y elimina el primer elemento por la izquierda o por la derecha.
- `blpop(claves, timeout=segundos)`: versión bloqueante de `lpop`. Si la lista está vacía, el script de Python se queda esperando hasta que entre un dato o se agote el tiempo (ideal para *workers* en segundo plano).
- `lrange(clave, inicio, fin)`: obtiene los elementos dentro de un rango de índices (`0` es el primero, `-1` es el último). No los elimina de la lista.
- `ltrim(clave, inicio, fin)`: recorta la lista para conservar **únicamente** los elementos dentro del rango indicado, borrando el resto.

Ejemplo práctico de una cola de envío de correos e historial de actividad limitada

```python
# --- CASO A: Cola de tareas (FIFO) ---
cola_emails = "tareas:envio_emails"

# 1. El servidor web encola tareas por la izquierda (lpush)
r.lpush(cola_emails, "bienvenida:ana@mail.com", "factura:carlos@mail.com", "reset_pwd:elena@mail.com")

# 2. Un proceso consumidor (worker) extrae la tarea más antigua por la derecha (rpop)
tarea_actual = r.rpop(cola_emails)
print(f"Procesando tarea prioritaria: {tarea_actual}")

# Consultar cuántas tareas quedan pendientes en la cola
print(f"Tareas restantes en la cola: {r.llen(cola_emails)}")


# --- CASO B: Historial de los últimos 3 inicios de sesión (Lista acotada) ---
historial_key = "usuario:88:ultimos_logins"

# Simulamos que el usuario inicia sesión desde distintos dispositivos
for evento in ["2026-10-01 (Web)", "2026-10-02 (App iOS)", "2026-10-05 (Tablet)", "2026-10-06 (SmartTV)"]:
    r.lpush(historial_key, evento)       # Insertamos el más reciente al principio (posición 0)
    r.ltrim(historial_key, 0, 2)         # Recortamos para quedarnos SOLO con los 3 primeros (índices 0, 1 y 2)

ultimos_accesos = r.lrange(historial_key, 0, -1)
print(f"\nÚltimos 3 accesos del usuario: {ultimos_accesos}")
```


### 6.6.2.- Tipo de dato: `Set` (Conjuntos)

Un `Set` es una colección **desordenada** de cadenas de texto **únicas** (no permite duplicados). Si intentas añadir el mismo elemento diez veces, solo se guardará una vez.

Su mayor potencia radica en que permite realizar **operaciones matemáticas de conjuntos** (intersección, unión y diferencia) directamente en memoria a gran velocidad. Son ideales para gestionar **etiquetas (*tags*)**, listas de control de acceso (IPs bloqueadas), seguimiento de elementos únicos o motores de emparejamiento ("amigos en común" o "usuarios que compraron A y también B").

Los principales métodos son:

- `sadd(clave, *miembros)`: añade uno o varios elementos al conjunto (ignora los que ya existen).
- `smembers(clave)`: devuelve un `set` de Python con todos los elementos del conjunto.
- `sismember(clave, miembro)`: comprueba en tiempo constante $O(1)$ si un elemento existe en el conjunto (`1`/`True` o `0`/`False`).
- `srem(clave, *miembros)`: elimina uno o varios elementos específicos.
- `sinter(claves)` / `sunion(claves)` / `sdiff(claves)`: calcula la intersección (elementos comunes), unión (todos sin repetir) o diferencia entre varios conjuntos.

Ejemplo práctico de un sistema de etiquetas y coincidencias entre usuarios

```python
# 1. Guardar las tecnologías que dominan dos candidatos (si hay repetidas, Redis las ignora)
r.sadd("candidato:ana:skills", "Python", "SQL", "Docker", "Python")
r.sadd("candidato:luis:skills", "Java", "Docker", "SQL", "Kubernetes")

# 2. Comprobar rápidamente si Ana sabe Docker
tiene_docker = r.sismember("candidato:ana:skills", "Docker")
print(f"¿Ana domina Docker?: {bool(tiene_docker)}")

# 3. Operaciones de conjuntos entre ambos candidatos
comunes = r.sinter("candidato:ana:skills", "candidato:luis:skills")
todas = r.sunion("candidato:ana:skills", "candidato:luis:skills")
solo_ana = r.sdiff("candidato:ana:skills", "candidato:luis:skills")

print(f"Habilidades en común (Intersección): {comunes}")
print(f"Habilidades totales del equipo (Unión): {todas}")
print(f"Habilidades que tiene Ana pero NO Luis (Diferencia): {solo_ana}")
```


### 6.6.3.- Tipo de dato: `Streams` (Flujos de Eventos)

Introducidos para funcionar como un registro de eventos de solo escritura (*append-only log*), los `Streams` son la alternativa ligera a herramientas como Apache Kafka dentro de Redis. A diferencia de las `Lists`, cuando un consumidor lee un mensaje de un `Stream`, **el mensaje no se borra**: queda registrado con un ID único basado en el tiempo (`timestamp-secuencia`) para que otros servicios también puedan leerlo o auditarlo en el futuro.

Se utilizan para **arquitecturas orientadas a eventos**, recolección de telemetría de dispositivos IoT, registros de auditoría centralizados y comunicación asíncrona entre microservicios.

Métodos en Python:

- `xadd(clave, diccionario_campos, maxlen=N)`: Añade un nuevo evento (en forma de diccionario) al final del flujo y devuelve su ID temporal autogenerado. Con `maxlen` puedes limitar el tamaño del histórico.
- `xrange(clave, min='-', max='+', count=N)`: Lee eventos dentro de un rango de IDs temporales (`'-'` es desde el principio, `'+'` hasta el final).
- `xread(streams={clave: ultimo_id}, count=N, block=ms)`: Lee eventos nuevos a partir de un ID concreto. Permite quedarse escuchando en tiempo real con `block`.

Veamos un ejemplo práctico de un registro centralizado de eventos de sensores (Telemetría):

```python
stream_key = "telemetria:planta_solar"

# 1. Dos sensores publican eventos en el Stream (Redis genera el ID timestamp automáticamente)
id_1 = r.xadd(stream_key, {"sensor_id": "inversor_01", "temperatura": 45.2, "estado": "OK"})
id_2 = r.xadd(stream_key, {"sensor_id": "inversor_02", "temperatura": 78.9, "estado": "ALERTA"})
id_3 = r.xadd(stream_key, {"sensor_id": "inversor_01", "temperatura": 46.0, "estado": "OK"})

print(f"Eventos registrados con IDs temporales: {id_1}, {id_2}, {id_3}")

# 2. Leer todos los eventos históricos desde el principio ('-') hasta el final ('+')
historial = r.xrange(stream_key, min='-', max='+')
print("\nHistorial completo del Stream:")
for event_id, datos in historial:
    print(f" [{event_id}] -> Sensor: {datos['sensor_id']} | Temp: {datos['temperatura']}°C ({datos['estado']})")

# 3. Leer únicamente los eventos ocurridos DESPUÉS del primer evento (id_1)
nuevos_eventos = r.xread(streams={stream_key: id_1}, count=10)
print(f"\nEventos posteriores a {id_1}: {len(nuevos_eventos[0][1])} encontrados.")
```


### 6.6.4.- Tipo de dDato: `Geospatial` (índices geoespaciales)

Redis permite almacenar coordenadas geográficas (**longitud y latitud**) y realizar cálculos espaciales complejos en tiempo real (Internamente los guarda como un `Sorted Set` codificando las coordenadas mediante un algoritmo *Geohash* de 52 bits).

Es la estructura clave para aplicaciones de movilidad, **búsqueda de puntos de interés cercanos** ("restaurantes o cajeros a menos de 2 km de mi ubicación") y cálculo de distancias en rutas de reparto.

Los métodos principales en Python son:

- `geoadd(clave, [longitud, latitud, miembro, ...])`: Registra uno o varios lugares con sus coordenadas. **Importante:** En Redis siempre va primero la **longitud** y luego la **latitud**.
- `geodist(clave, miembro1, miembro2, unit='km')`: Calcula la distancia en línea recta entre dos puntos (`'m'`, `'km'`, `'mi'`, `'ft'`).
- `geosearch(clave, longitude=lon, latitude=lat, radius=r, unit='km', withdist=True)`: Busca todos los puntos almacenados que estén dentro de un radio alrededor de unas coordenadas dadas.

En el siguiente código puedes ver un ejemplo de un localizador de tiendas cercanas y cálculo de distancias:

```python
geo_key = "logistica:almacenes_espana"

# 1. Registrar las coordenadas de varios almacenes [Longitud, Latitud, Nombre]
r.geoadd(geo_key, [
    -3.7038, 40.4168, "Almacen_Madrid",
    -0.3763, 39.4699, "Almacen_Valencia",
    -5.9845, 37.3891, "Almacen_Sevilla",
    -3.6883, 40.4530, "Almacen_Madrid_Norte"
])

# 2. Calcular la distancia en kilómetros entre el almacén de Madrid y el de Valencia
distancia = r.geodist(geo_key, "Almacen_Madrid", "Almacen_Valencia", unit="km")
print(f"Distancia entre Madrid y Valencia: {distancia:.2f} km")

# 3. Un cliente está en Atocha (Lon: -3.6919, Lat: 40.4065) y busca almacenes en un radio de 10 km
cercanos = r.geosearch(
    geo_key,
    longitude=-3.6919,
    latitude=40.4065,
    radius=10,
    unit="km",
    withdist=True,
    sort="ASC" # Ordenar del más cercano al más lejano
)

print("\nAlmacenes a menos de 10 km del cliente:")
for nombre, dist in cercanos:
    print(f" -> {nombre} está a {dist:.2f} km")
```



### 6.6.5.- Estructuras probabilísticas y de bits: `HyperLogLog` y `Bitmaps`

Cuando el volumen de datos escala a cientos de millones de registros, guardar cada elemento individualmente consume demasiada memoria RAM. Redis ofrece dos estructuras ultraligeras para analítica masiva:

#### A. `HyperLogLog` (Conteo de elementos únicos a gran escala)

Permite estimar la **cardinalidad** (el número de elementos únicos sin duplicados) de un conjunto gigante utilizando **como máximo 12 KB de memoria**, con un margen de error estándar del 0.81%. A diferencia de un `Set`, no puedes recuperar los elementos que metiste, solo saber cuántos distintos hubo.

Ejemplo de un caso de uso en el que se cuenta visitantes únicos (IPs o `user_id`) de un portal web de alto tráfico al mes o búsquedas únicas diarias sin agotar la RAM.

```python
hll_key = "analitica:visitas_unicas:2026-11-05"

# 1. Añadir elementos con pfadd (aunque se repitan, solo cuentan una vez)
r.pfadd(hll_key, "ip_192.168.1.1", "ip_10.0.0.1", "ip_192.168.1.1", "ip_172.16.0.5")

# 2. Obtener el conteo aproximado de elementos únicos con pfcount
unicos = r.pfcount(hll_key)
print(f"Visitantes únicos hoy (HyperLogLog): {unicos}")

```

#### B. `Bitmaps` (mapas de bits)

No es un tipo de dato independiente, sino un conjunto de operaciones a nivel de bits (`0` o `1`) sobre un `String`. Permite almacenar estados booleanos (Sí/No, Activo/Inactivo) para millones de usuarios numerados por ID ocupando apenas unos pocos Megabytes (1 millón de bits = ~125 KB).

Ejemplo de un control de asistencia diaria de usuarios, saber qué usuarios han aceptado los nuevos términos legales o *feature flags* por ID de usuario.

```python
bitmap_key = "asistencia:2026-11-05"

# 1. Marcar con un 1 (True) que los usuarios con ID 5, 42 y 1000 se han conectado hoy
r.setbit(bitmap_key, 5, 1)
r.setbit(bitmap_key, 42, 1)
r.setbit(bitmap_key, 1000, 1)

# 2. Consultar si un usuario concreto se conectó hoy (devuelve 1 o 0)
conectado_42 = r.getbit(bitmap_key, 42)
conectado_99 = r.getbit(bitmap_key, 99)
print(f"¿Se conectó el usuario 42?: {bool(conectado_42)} | ¿Y el usuario 99?: {bool(conectado_99)}")

# 3. Contar el total de usuarios activos en el día (cuenta cuántos bits están a 1)
total_activos = r.bitcount(bitmap_key)
print(f"Total de usuarios activos hoy (Bitmap): {total_activos}")
```


## 6.7.- Comandos útiles de limpieza y depuración para el laboratorio

- `r.exists("clave")`: devuelve `1` si la clave existe o `0` si no existe.
- `r.delete("clave1", "clave2")`: elimina una o varias claves de cualquier tipo.
- `r.keys("patron:*")`: busca claves que coincidan con un patrón (ej. `r.keys("features:cliente:*")`). *Útil en el laboratorio, pero prohibido en entornos de producción con millones de claves porque bloquea el servidor.*
- `r.flushdb()`: **borra absolutamente todas las claves** de la base de datos actual. Ideal para reiniciar el entorno de pruebas desde cero.