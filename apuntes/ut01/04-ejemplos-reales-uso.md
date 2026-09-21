```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción al Big Data
Apartado:                   4.- Ejemplos reales de uso de Big Data
Resultados de aprendizaje:  ?
```


# UT01. INTRODUCCIÓN AL BIG DATA


## 4. Ejemplos reales de uso de Big Data

### 4.1. Netflix: Personalización extrema y producción de contenidos

Netflix aprovecha los datos generados por más de 230 millones de suscriptores para maximizar la retención de clientes y optimizar sus inversiones en catálogo. 

Captura datos como hábitos de consumo (tiempo de sesión, dispositivos, fechas y horarios), interacciones (pausas, rebobinados, abandonos a mitad de título, búsquedas) o datos contextuales (ubicación, idioma y perfiles de visualización).




| **TÉCNICAS ALGORÍTMICAS Y EXPERIMENTACIÓN** |  |
| **Filtrado colaborativo y Machine Learning** | Comparación del comportamiento del usuario con perfiles de gustos similares y análisis de metadatos de las obras (género, ritmo narrativo, reparto). |
| **Personalización visual** | Generación y asignación de miniaturas dinámicas según los intereses del usuario (destacando a un actor concreto o acentuando el componente de acción o romance). |
| **Pruebas A/B sistemáticas (*Split Testing*)** | Evaluación paralela de interfaces, posiciones de botones, textos y versiones de algoritmos para determinar empíricamente qué opción maximiza el tiempo de visualización. |
| **Producción basada en datos (*Data-Driven Content*)** | La creación de la serie *House of Cards* (2013) supuso una inversión pionera de 100 millones de dólares decidida tras constatar en los datos la popularidad de la serie británica original, el alto seguimiento del director David Fincher, el interés por los dramas políticos y las búsquedas del actor Kevin Spacey; un modelo predictivo repetido en éxitos como *Stranger Things*, *Gambito de dama* o *A ciegas*. |



#### Aplicación de las 5 V's en Netflix

- **Volumen:** Cientos de petabytes procedentes de la actividad audiovisual de más de 230 millones de usuarios.
- **Velocidad:** Ingesta y análisis en tiempo real para modificar recomendaciones de forma dinámica mientras el usuario navega.
- **Variedad:** Fusión de datos estructurados (calificaciones, metadatos), semiestructurados (logs de clics) y no estructurados (fotogramas, comentarios en redes sociales).
- **Veracidad:** Filtrado continuo de sesiones inactivas, accesos accidentales o cuentas compartidas para evitar distorsiones en los modelos.
- **Valor:** Reducción de la tasa de cancelación (*churn*) y garantía de éxito en nuevas inversiones. Se estima que su motor de recomendación ahorra más de 1.000 millones de dólares anuales en retención, influyendo en más del 80% de los títulos reproducidos mediante la categorización en más de 2.000 microgéneros.


#### Técnicas algorítmicas y experimentación

- **Filtrado colaborativo y Machine Learning:** Comparación del comportamiento del usuario con perfiles de gustos similares y análisis de metadatos de las obras (género, ritmo narrativo, reparto).
- **Personalización visual:** Generación y asignación de miniaturas dinámicas según los intereses del usuario (destacando a un actor concreto o acentuando el componente de acción o romance).
- **Pruebas A/B sistemáticas (*Split Testing*):** Evaluación paralela de interfaces, posiciones de botones, textos y versiones de algoritmos para determinar empíricamente qué opción maximiza el tiempo de visualización.
- **Producción basada en datos (*Data-Driven Content*):** La creación de la serie *House of Cards* (2013) supuso una inversión pionera de 100 millones de dólares decidida tras constatar en los datos la popularidad de la serie británica original, el alto seguimiento del director David Fincher, el interés por los dramas políticos y las búsquedas del actor Kevin Spacey; un modelo predictivo repetido en éxitos como *Stranger Things*, *Gambito de dama* o *A ciegas*.


### 4.2. Google Maps: Tráfico en tiempo real y rutas inteligentes

Google Maps da servicio a más de 1.000 millones de usuarios activos mensuales manteniendo un 99,9% de disponibilidad.

#### Aplicación de las 5 V's en Google Maps

- **Volumen:** Emisión masiva y continua de coordenadas y telemetría desde cientos de millones de terminales móviles.
- **Velocidad:** Tiempos de respuesta y recálculo inferiores a un segundo para detectar incidentes y adaptar las rutas en marcha.
- **Variedad:** Integración de más de 15 fuentes heterogéneas: GPS móvil, sensores viales IoT, cámaras de tráfico, imágenes satelitales y cartografía oficial.
- **Veracidad:** Filtrado algorítmico de errores de precisión satelital y lecturas erráticas en túneles o vías secundarias.
- **Valor:** Reducción del 20% en retrasos imprevistos, ahorro de un 22% en tiempos de viaje y reducción de 2,1 millones de toneladas anuales de emisiones de $\text{CO}_2$ a nivel global.

#### Funcionalidades avanzadas

- **Predicción de tráfico y tiempo estimado de llegada (ETA):** Modelos de regresión que cruzan el histórico de la vía, obras, fases semafóricas y el comportamiento de la flota en tiempo real (si el 70% de vehículos reducen velocidad, se infiere una retención), logrando precisiones del 97% en grandes ciudades.
- **Rutas ecológicas:** Combinación de datos topográficos (pendientes), patrones de frenado y fluidez para trazar itinerarios que minimizan el consumo de combustible.
- **Modo Crisis ante catástrofes:** Detección y activación automática ante terremotos, incendios o conflictos bélicos integrando alertas oficiales de protección civil y anomalías de movimiento poblacional para guiar a los ciudadanos hacia refugios, pasos fronterizos seguros y hospitales operativos.
- **Privacidad y aprendizaje de hábitos:** Identificación de trayectos rutinarios del usuario para silenciar alertas innecesarias y notificar únicamente incidencias anómalas en la ruta.



### 4.3. Amazon: Recomendación predictiva y logística

Amazon gestiona su operativa mundial sobre un *Data Lake* que procesa más de 1,5 petabytes de información.

#### Aplicación de las 5 V's en Amazon

- **Volumen:** Millones de transacciones por segundo, clics de navegación, reseñas de producto y catálogo global.
- **Velocidad:** Generación de recomendaciones en milisegundos y optimización de rutas para posibilitar entregas en menos de 24 horas (Amazon Prime).
- **Variedad:** Fusión de historiales de compra, listas de deseos, devoluciones, condiciones meteorológicas y telemetría de furgonetas.
- **Veracidad:** Detección de reseñas fraudulentas, eliminación de duplicados y conciliación de stock en tiempo real.
- **Valor:** El motor de recomendaciones impulsa aproximadamente el 35% de las ventas totales de la plataforma y optimiza la rentabilidad operativa.

#### Casos de uso destacados

- **Previsión de demanda e inventario:** Análisis de estacionalidad, noticias y tendencias para ubicar físicamente los productos en los centros logísticos más cercanos antes de que el usuario efectúe la compra.
- **Precios dinámicos:** Algoritmos que modifican los precios de millones de artículos varias veces al día atendiendo al stock disponible, precios de la competencia y elasticidad de la demanda.
- **Prevención del fraude:** Monitorización en tiempo real de anomalías en los patrones de compra y movimientos atípicos en medios de pago.



### 4.4. Sistema sanitario y datos epidemiológicos

El sector de la salud procesa desde terabytes hasta petabytes de datos clínicos para transformar la medicina reactiva en preventiva y personalizada.

#### Aplicación de las 5 V's en Sanidad

- **Volumen:** Historiales clínicos electrónicos de millones de pacientes, secuencias genéticas completas e imágenes radiológicas.
- **Velocidad:** Detección de brotes infecciosos en tiempo casi real y gestión inmediata de ocupación de camas de UCI.
- **Variedad:** Datos tabulares estructurados (analíticas de laboratorio), semiestructurados (formularios y encuestas de salud) y no estructurados (resonancias magnéticas, notas médicas en texto libre, señales de pulsioxímetros).
- **Veracidad:** Aplicación de estándares internacionales estrictos de interoperabilidad y calidad de datos (como **HL7** y **FHIR**) para evitar falsos positivos y duplicidades de pacientes.
- **Valor:** Diagnósticos precoces, optimización del gasto hospitalario y diseño de terapias a medida que salvan vidas.


#### Aplicaciones prácticas

- **Medicina de precisión y genómica:** Identificación de biomarcadores moleculares para anticipar la respuesta a fármacos en oncología.
- **Diagnóstico asistido por IA:** Redes neuronales aplicadas a la detección precoz de tumores en radiología o diagnóstico de patologías raras.
- **Vigilancia epidemiológica y prevención remota:** Modelos de propagación de enfermedades infecciosas y monitorización continua mediante *wearables* que registran constantes vitales (ritmo cardíaco, saturación, glucosa) de forma no invasiva.



### 4.5. Smart Cities (Ciudades Inteligentes)

Las ciudades inteligentes capturan flujos masivos de datos para garantizar la sostenibilidad ambiental y mejorar la calidad de vida urbana.

#### Aplicación de las 5 V's en Smart Cities

- **Volumen:** Petabytes diarios originados en redes de transporte, contadores de consumo, cámaras urbanas y sensores ambientales.
- **Velocidad:** Actuación en segundos para gestionar atascos, picos de polución o emergencias urbanas.
- **Variedad:** Señales estructuradas de redes eléctricas inteligentes (*smart grids*), partes semiestructurados de incidencias y secuencias de vídeo/audio de videovigilancia.
- **Veracidad:** Filtrado de fallos en sensores a la intemperie y validación de avisos vecinales.
- **Valor:** Disminución del gasto municipal, reducción de la huella de carbono y mayor eficiencia en servicios esenciales.


#### Ejemplos de implementación urbana

- **Gestión de tráfico y transporte:** Sincronización dinámica de fases semafóricas según la densidad del flujo y reprogramación de frecuencias de autobuses en tiempo real.
- **Alumbrado inteligente:** Farolas con sensores que regulan automáticamente la intensidad lumínica según la luz ambiental y el tránsito peatonal (ejemplo implementado en Ámsterdam).
- **Gestión eficiente de residuos:** Sensores de volumen en contenedores que recalculan diariamente las rutas óptimas de los camiones de recogida para reducir combustible y personal (ejemplo implementado en Santander).
- **Control ambiental y restricciones:** Redes de monitorización de calidad del aire que activan limitaciones dinámicas al tráfico de vehículos contaminantes al superar umbrales de $\text{NO}_2$ (como en París).
- **Seguridad y turismo inteligente:** Detección de incidentes en vía pública mediante analítica de vídeo y redistribución de flujos turísticos hacia zonas descongestionadas.


---

[Volver al índice](ut01_index.md)