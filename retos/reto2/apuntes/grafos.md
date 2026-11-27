```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT03. Grafos, algoritmia y redes complejas
Apartado:                   1.- Conceptos importantes sobre grafos
Resultados de aprendizaje:  ?
```

# 1.- Conceptos sobre grafos

## 1.1.- Definición y anatomía del grafo

Un **grafo** es una estructura matemática compuesta por un conjunto de entidades conectadas entre sí, definida formalmente como G = (V, E), donde:

- **V (Vértices o Nodos):** representan los actores del ecosistema. En nuestro reto, son las carteras de usuarios (`:Wallet`) y las casas de cambio (`:Exchange`).
- **E (Aristas, Enlaces o Relaciones):** representan las interacciones entre los nodos. En nuestro caso, son las transferencias de criptoactivos (`[:TRANSFIERE]`).



### Tipología del grafo del proyecto

Para modelar correctamente el flujo de dinero en *NovaSwap*, nuestro grafo posee tres características estructurales clave:

1. **Grafo dirigido (Digrafo):** un grafo dirigido es aquel en el que las aristas tienen una orientación estricta (u -> v). Que la cartera A envíe fondos a B no implica que B envíe fondos a A. En NetworkX se representa mediante la clase `nx.DiGraph()`.
2. **Grafo ponderado:** cuando hablamos de grafos ponderados cada relación tiene asociado un valor numérico o **peso** (*weight*) que cuantifica la intensidad de la conexión. En nuestro dominio, el peso principal es el volumen financiero transferido (`monto_btc`).
3. **Multigrafo de propiedades (Labeled Property Graph - LPG):**
    - **En Neo4j (Multigrafo):** entre dos mismas carteras A y B pueden existir múltiples transacciones independientes en distintos momentos (`tx_hash` diferentes). Además, tanto nodos como relaciones almacenan diccionarios de atributos (ej. `kyc`, `pais_registro`, `timestamp`, `fee_btc`) y los nodos se agrupan mediante etiquetas (`:Wallet`, `:Exchange`).
    - **En NetworkX (Agregación a Grafo Simple Dirigido):** al construir `nx.DiGraph()` en el Módulo 3, colapsamos las múltiples transferencias entre un mismo par (u, v) en una única arista dirigida cuyo atributo `weight` es la suma total de BTC enviados de u a v.







## 1.2. Conectividad, grados y flujo en Nodos

En un grafo dirigido, la actividad de cualquier nodo se analiza desglosando sus conexiones entrantes y salientes:

- **Grado de Entrada:** número de aristas que llegan al nodo (transacciones recibidas).
- **Grado de Salida:** número de aristas que parten del nodo (transacciones enviadas).
- **Grado Ponderado:** en lugar de contar transacciones, suma los pesos de las aristas. En nuestro análisis forense (Módulo 3) distinguimos entre:
    - **Flujo de entrada:** suma del importe de todas las aristas entrantes.
    - **Flujo de salida:** suma del importe de todas las aristas salientes.

Estos valores son importantes para nuestro análisis forense ya que nos ayudará a detectar **nodos de paso** y **sumideros**:

- Una cartera que actúa como **mula o *mixer*** tiene un balance equilibrado: su entrada/salida (el flujo de entrada dividido entre el de salida) es muy cercano a `1.0`.
- Un **Exchange No-KYC** suele actuar como **sumidero**, es decir, tiene un alto grado y volumen de entrada, pero un flujo de salida nulo o irrelevante hacia el resto de la red investigada, ya que es el punto donde los fondos salen del circuito hacia dinero fíat.



## 1.3.- Caminos, distancias y topologías de lavado

El rastreo de fondos consiste en recorrer secuencias de nodos conectados siguiendo la dirección de las flechas:

- **Camino dirigido:** secuencia ordenada de nodos tal que existe una arista dirigida entre dos nodos en cada paso.
- **Longitud del camino:** número de aristas que componen un camino. Por ejemplo, si A -> B -> C, la distancia es de 2 saltos.
- **Camino más corto:** el camino dirigido entre un nodo origen y un nodo destino que requiere el menor número de saltos. En el Módulo 2 (`shortestPath`), sirve para medir la cercanía mínima entre una cartera sospechosa y una vía de escape (`Exchange` con `kyc = false`).
- **Ciclo dirigido:** un camino cerrado donde el nodo inicial y el nodo final son el mismo sin repetir nodos intermedios. Es la base estructural del ***Wash Trading*** (anillos de 3 a 6 saltos que simulan volumen legítimo devolviendo el dinero al punto de partida).



### Patrones topológicos específicos del reto

| Patrón Forense              | Estructura Topológica                                                                                                                             | Propósito en el Blanqueo                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Wash Trading** | **Ciclo simple:** A -> B -> C -> D -> A (3 a 6 saltos).       | Generar ruido transaccional y ofuscar el origen haciendo circular el mismo capital entre cuentas controladas. |
| **Smurfing**     | **Bipartito / Diamante (*Fan-out / Fan-in*):** un origen dispersa fondos a más de 5 nodos intermedios (mulas) que convergen en un único destino. | Evitar alertas por importes elevados fraccionando una suma grande en múltiples micro-transacciones paralelas. |
| **Peeling Chain** | **Cadena lineal con ramificaciones:** una ruta troncal (>= 4 saltos) donde en cada nodo avanza < 80\% del capital y se desvía < 15\% hacia un `:Exchange`. | Liquidar grandes botines poco a poco en casas de cambio sin levantar sospechas por retiradas masivas de golpe. |



## 1.4.- Métricas de centralidad y análisis estructural

### 1.4.1.- Intermediación (*Betweenness Centrality*)

La **centralidad de intermediación** (*Betweenness Centrality*) es una métrica que sirve para medir qué tan indispensable es un intermediario dentro de una red (por ejemplo, una red de transferencias de criptomonedas o comunicaciones).

Para entenderlo, imagina que la red es un mapa de carreteras y cada nodo es una ciudad o un cruce:

- **¿Qué mide exactamente?** Cuenta cuántas veces un punto específico actúa como parada obligatoria en las rutas más directas (o eficientes) que conectan al resto de los puntos entre sí. Si para ir de la ciudad A a la ciudad C el camino más rápido siempre obliga a pasar por la ciudad B, entonces B tiene una intermediación muy alta.
- **Aplicación forense** (rastreo de fondos): en el análisis de transacciones ilícitas, un nodo (como una cartera digital o un servicio mezclador) con una intermediación muy alta funciona como un puente crítico o cuello de botella. No tiene por qué ser el que más dinero acumula, pero sí es el paso obligado por donde fluye el dinero para moverse de un lado a otro.
- **Impacto estratégico**: identificar estos nodos clave permite a las autoridades atacar el punto más vulnerable de la red. Al bloquear o intervenir ese único "puente", se cortan de golpe múltiples rutas que los delincuentes usan para llevar el dinero hacia los exchanges donde intentan retirarlo o blanquearlo.

Un nodo con un *Betweenness* elevado (por encima del percentil 90, como exige la **Hipótesis H_1**) actúa como un **cuello de botella o puente crítico**. Si las autoridades bloquean este nodo (*Takedown* en el Módulo 4), se interrumpen multitud de rutas de escape hacia los exchanges.


### 1.4.2.- PageRank ponderado

Algoritmo que mide la importancia o prestigio de un nodo basándose en la estructura de sus enlaces entrantes. En nuestra versión ponderada por `weight` (`monto_btc`):

1. Un nodo tiene mayor PageRank si recibe transferencias de muchos nodos.
2. Su puntuación sube aún más si recibe fondos de nodos que, a su vez, mueven grandes volúmenes de BTC.

En nuestro prooyecto en concreto nos sirve para identificar los **grandes concentradores de capital** de la red (tanto grandes exchanges como carteras principales de acumulación del botín).



## 1.5. Subgrafos, componentes y resiliencia (Takedown)

Para el análisis de impacto, es fundamental comprender cómo se fragmenta una red al eliminar vértices:

- **Subgrafo inducido:** un subconjunto de nodos del grafo original junto con todas las aristas que conectan exclusivamente a dichos nodos entre sí.
- **Alcanzabilidad (*Reachability*):** se dice que un destino `t` es alcanzable desde un origen `s` si existe al menos un camino dirigido entre `s` y `t`.
- **Percolación / ataque dirigido (*Node Removal*):** proceso de eliminar de la topología los nodos con mayor centralidad (los 5 nodos con mayor *Betweenness*). Al borrar un nodo `v` en NetworkX (`G.remove_nodes_from`), desaparecen también todas sus aristas incidentes (entrantes y salientes), lo que puede romper la alcanzabilidad entre las carteras sospechosas y los `Exchanges No-KYC`.


## 1.6. Flujo, tetención temporal y criterio empírico (Hipótesis H1)

En el análisis forense de criptoactivos, la topología estática no basta: un nodo puede tener conexiones intermedias o un *Betweenness* alto por motivos legítimos (por ejemplo, un *hub* de distribución de pagos o un servicio con tráfico normal). Para aislar carteras que actúan como **nodos puente o *mixers* de paso rápido**, el Módulo 3 combina **estructura de grafo** con **dinámica temporal y volumétrica**.

### 1.6.1. Dinámica de flujo (`btc_recibido`, `btc_enviado` y `ratio_flujo`)

A partir del historial de transferencias de cada cartera se cuantifica el volumen que entra y sale del nodo:

- **Volumen recibido (`btc_recibido`):** suma total de `monto_btc` en las transacciones entrantes hacia la cartera.
- **Volumen enviado (`btc_enviado`):** suma total de `monto_btc` en las transacciones salientes desde la cartera.
- **Ratio de flujo (`ratio_flujo`):** cociente entre el volumen enviado y el volumen recibido (o `0.0` si `btc_recibido == 0`): *Ratio = btc_enviado / btc_recibido*.
- **Interpretación del rango `[0.95, 1.05]`:** un valor próximo a `1.0` indica que la cartera reenvía prácticamente la totalidad de los fondos que recibe sin acumular saldo residual relevante en ese periodo, comportamiento característico de carteras de paso, *mixers* o nodos intermedios de canalización.



### 1.6.2. Tiempo de retención (`retencion_media_min`)

El análisis temporal mide cuánto permanecen los fondos en una cartera antes de volver a moverse hacia otra dirección, calculando en minutos el intervalo entre la recepción de fondos y las salidas posteriores asociadas a ese nodo.

- **Interpretación del umbral `< 30` minutos:** una permanencia media baja indica un **tránsito rápido** (o automatizado) de los fondos. En un escenario de lavado o dispersión rápida, las carteras puente suelen reexpedir el capital con poca demora para dificultar el seguimiento y encadenar saltos antes de una posible congelación o retirada.



### 1.6.3. Percentiles y selección por umbral en la Hipótesis H_1

En estadística descriptiva y análisis de redes, el **percentil 90** de una distribución (como `betweenness` en el grafo) es el valor por debajo del cual se encuentra el 90 % de las observaciones. Exigir `betweenness > percentil 90` implica seleccionar únicamente el **10 % superior** de nodos con mayor centralidad de intermediación en la red.

La **Hipótesis H_1** exige que un nodo de tipo `:Wallet` cumpla **simultáneamente** tres condiciones para ser considerado un candidato prioritario a nodo puente de paso rápido:

| Condición en $H_1$         | Definición formal                | Qué descarta / Qué selecciona                                                                                             |
| -------------------------- | -------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| **1. Intermediación alta** | `betweenness > P90(betweenness)` | Filtra nodos periféricos o de paso trivial; conserva el 10 % con mayor peso como puente en rutas cortas.                  |
| **2. Paso equilibrado**    | `0.95 <= ratio_flujo <= 1.05`    | Descarta sumideros puros o acumuladores que retienen el saldo; selecciona carteras que reenvían casi todo lo que reciben. |
| **3. Baja retención**      | `retencion_media_min < 30`       | Descarta movimientos lentos o diferidos; prioriza carteras con rotación rápida de fondos (menos de 30 minutos de media).  | 


**Nota para el informe:** cumplir H_1 no constituye por sí solo una prueba definitiva de autoría, sino un **criterio de priorización empírico**. Al cruzar las carteras resultantes de H_1 (`docs/nodos_puente_detectados.csv`) con los hallazgos del Módulo 2 (ciclos de *wash trading*, *smurfing* y rutas hacia `Exchanges No-KYC`), podrás verificar qué nodos puente coinciden con estructuras de blanqueo ya identificadas y cuáles representan pistas adicionales que conviene revisar.

