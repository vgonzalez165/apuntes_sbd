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

Un **grafo** es una estructura matemática compuesta por un conjunto de entidades conectadas entre sí, definida formalmente como $G = (V, E)$, donde:

- **$V$ (Vértices o Nodos):** representan los actores del ecosistema. En nuestro reto, son las carteras de usuarios (`:Wallet`) y las casas de cambio (`:Exchange`).
- **$E$ (Aristas, Enlaces o Relaciones):** representan las interacciones entre los nodos. En nuestro caso, son las transferencias de criptoactivos (`[:TRANSFIERE]`).



### Tipología del grafo del proyecto

Para modelar correctamente el flujo de dinero en *NovaSwap*, nuestro grafo posee tres características estructurales clave:

1. **Grafo dirigido (Digrafo):** un grafo dirigido es aquel en el que las aristas tienen una orientación estricta ($u \to v$). Que la cartera $A$ envíe fondos a $B$ no implica que $B$ envíe fondos a $A$. En NetworkX se representa mediante la clase `nx.DiGraph()`.
2. **Grafo ponderado:** cuando hablamos de grafos ponderados cada relación tiene asociado un valor numérico o **peso** (*weight*) que cuantifica la intensidad de la conexión. En nuestro dominio, el peso principal es el volumen financiero transferido (`monto_btc`).
3. **Multigrafo de propiedades (Labeled Property Graph - LPG):**
    - **En Neo4j (Multigrafo):** entre dos mismas carteras $A$ y $B$ pueden existir múltiples transacciones independientes en distintos momentos (`tx_hash` diferentes). Además, tanto nodos como relaciones almacenan diccionarios de atributos (ej. `kyc`, `pais_registro`, `timestamp`, `fee_btc`) y los nodos se agrupan mediante etiquetas (`:Wallet`, `:Exchange`).
    - **En NetworkX (Agregación a Grafo Simple Dirigido):** al construir `nx.DiGraph()` en el Módulo 3, colapsamos las múltiples transferencias entre un mismo par $(u, v)$ en una única arista dirigida cuyo atributo `weight` es la suma total de BTC enviados de $u$ a $v$.







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
- **Longitud del camino:** número de aristas que componen un camino. Por ejemplo, si $A \to B \to C$, la distancia es de 2 saltos.
- **Camino más corto:** el camino dirigido entre un nodo origen y un nodo destino que requiere el menor número de saltos. En el Módulo 2 (`shortestPath`), sirve para medir la cercanía mínima entre una cartera sospechosa y una vía de escape (`Exchange` con `kyc = false`).
- **Ciclo dirigido:** un camino cerrado donde el nodo inicial y el nodo final son el mismo sin repetir nodos intermedios. Es la base estructural del ***Wash Trading*** (anillos de 3 a 6 saltos que simulan volumen legítimo devolviendo el dinero al punto de partida).



### Patrones topológicos específicos del reto

| Patrón Forense              | Estructura Topológica                                                                                                                             | Propósito en el Blanqueo                                                                                      |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Wash Trading** (Módulo 2) | **Ciclo simple:** $A \to B \to C \to D \to A$ (3 a 6 saltos).                                                                                     | Generar ruido transaccional y ofuscar el origen haciendo circular el mismo capital entre cuentas controladas. |
| **Smurfing** (Módulo 2)     | **Bipartito / Diamante (*Fan-out / Fan-in*):** Un origen dispersa fondos a $N \ge 5$ nodos intermedios (mulas) que convergen en un único destino. |                                                                                                               |

 | Evitar alertas por importes elevados fraccionando una suma grande en múltiples micro-transacciones paralelas.

 |
| **Peeling Chain** (Módulo 4) | **Cadena lineal con ramificaciones:** Una ruta troncal ($\ge 4$ saltos) donde en cada nodo avanza $\ge 80\%$ del capital y se desvía $\le 15\%$ hacia un `:Exchange`.

 | Liquidar grandes botines poco a poco en casas de cambio sin levantar sospechas por retiradas masivas de golpe.

 |

---

## 4. Métricas de Centralidad y Análisis Estructural

Cuando el grafo crece, buscar patrones fijos no basta; necesitamos métricas globales que midan la importancia estructural de cada nodo dentro de la red completa:

### 4.1. Intermediación (*Betweenness Centrality*)

Mide con qué frecuencia un nodo aparece en los **caminos más cortos** entre todos los demás pares de nodos del grafo. Se define para un nodo $v$ como:

$$C_B(v) = \sum_{s \neq v \neq t} \frac{\sigma_{st}(v)}{\sigma_{st}}$$

Donde $\sigma_{st}$ es el número total de caminos más cortos desde el nodo $s$ hasta el nodo $t$, y $\sigma_{st}(v)$ es el número de esos caminos que pasan obligatoriamente por $v$.

* **Interpretación forense:** Un nodo con un *Betweenness* elevado (por encima del percentil 90, como exige la **Hipótesis $H_1$**) actúa como un **cuello de botella o puente crítico**. Si las autoridades bloquean este nodo (*Takedown* en el Módulo 4), se interrumpen multitud de rutas de escape hacia los exchanges.



### 4.2. PageRank Ponderado

Algoritmo que mide la importancia o prestigio de un nodo basándose en la estructura de sus enlaces entrantes. En nuestra versión ponderada por `weight` (`monto_btc`):

1. Un nodo tiene mayor PageRank si recibe transferencias de muchos nodos.


2. Su puntuación sube aún más si recibe fondos de nodos que, a su vez, mueven grandes volúmenes de BTC.



* **Interpretación forense:** Identifica los **grandes concentradores de capital** de la red (tanto grandes exchanges como carteras principales de acumulación del botín).



---

## 5. Subgrafos, Componentes y Resiliencia (Takedown)

Para el análisis de impacto (Módulo 4), es fundamental comprender cómo se fragmenta una red al eliminar vértices:

* **Subgrafo inducido:** Un subconjunto de nodos del grafo original junto con todas las aristas que conectan exclusivamente a dichos nodos entre sí.
* **Alcanzabilidad (*Reachability*):** Se dice que un destino $t$ es alcanzable desde un origen $s$ si existe al menos un camino dirigido $s \leadsto t$.
* **Percolación / Ataque dirigido (*Node Removal*):** Proceso de eliminar de la topología los nodos con mayor centralidad (los 5 nodos con mayor *Betweenness*). Al borrar un nodo $v$ en NetworkX (`G.remove_nodes_from`), desaparecen también todas sus aristas incidentes (entrantes y salientes), lo que puede romper la alcanzabilidad entre las carteras sospechosas y los `Exchanges No-KYC`.



---

Si te encaja este nivel de profundidad y enfoque para tus alumnos, el siguiente paso lógico sería preparar el bloque de **Neo4j y Cypher** (modelo de propiedades, restricciones/índices, carga masiva con `UNWIND` + `MERGE` y patrones de búsqueda de longitud variable como `-[:TRANSFIERE*3..6]->` y `shortestPath`). ¿Continuamos con ese bloque?