```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT02. Persistencia documental, caché y procesamiento ETL
Reto:                       1. Sistema de triaje y trazabilidad de urgencias hospitalarias
Duración:                   7 semanas (14 horas presenciales + trabajo en equipo)
Resultados de aprendizaje:  RA1 (Criterios c, d, f). Integración de fuentes heterogéneas, construcción de datos complejos y selección de arquitecturas NoSQL.
                            RA3 (Criterios a, b, d). Extracción y almacenamiento políglota, eficiencia en valor analítico, contenerización y seguridad/normativa de datos.
```


# EVALUACIÓN


### 1. Rotación de roles (7 semanas)

Todos los alumnos del equipo rotarán en los diferentes roles de Scrum en **bloques de 2 semanas**, reservando la **Semana 7** para la consolidación, demo y pruebas finales.

En **grupos de 3 alumnos** hay 1 Product Owner, 1 Scrum Master y 1 Developer principal (aunque PO y SM también realizan en sus tareas asignadas). En **grupos de 4 alumnos** se añade un tercer rol técnico de liderazgo: **QA / DevOps Lead** (responsable de la reproducibilidad del entorno, pruebas de volumen y gestión de ramas en Git)

Las responsabilidades concretas de cada rol son:


| Rol | Cometido | Responsabilidades | Memoria |
| --- | -------- | ----------------- | ------- |
| **Product Owner** | Asegurar que el desarrollo responde a los requisitos clínicos y técnicos del hospital *(el qué y para qué)* | • Desglosar los requisitos técnicos en *User Stories* claras en el tablero.<br>• Redactar los **criterios de aceptación** de cada tarea antes de que empiece su desarrollo.<br>• Priorizar el Backlog al inicio de cada sprint, decidiendo qué entra y qué queda fuera.<br>• Validar si una funcionalidad cumple la *Definition of Done* (DoD) antes de permitir su cierre en el tablero.<br>• Actuar de interlocutor con el profesor ante dudas de alcance o lógica de negocio. | • Enlaces a 2 o 3 *User Stories* redactadas con criterios de aceptación explícitos.<br>• Historial de aprobación o rechazo de tareas en el tablero Kanban.<br>• Justificación documentada de las decisiones de priorización tomadas durante su turno. |
| **Scrum Master** | Facilitar la agilidad del equipo, vigilar el flujo continuo de trabajo y eliminar cualquier impedimento técnico u organizativo *(el cómo y el ritmo)* | • Moderar la sincronización semanal (5 minutos al inicio de cada sesión de 2 horas).<br>• Mantener actualizado el **tablero Kanban**, vigilando los límites de trabajo en curso y detectando tareas atascadas en *In Progress*.<br>• Detectar y desbloquear incidencias técnicas u organizativas que frenen a sus compañeros.<br>• Dinamizar la retrospectiva breve al cierre de sprint para consensuar compromisos de mejora entre bloques. | • Capturas comparativas del tablero Kanban (estado inicial vs. final de su periodo).<br>• Registro documentado de al menos un bloqueo técnico o de coordinación resuelto.<br>• Breve acta de acuerdos y mejoras alcanzadas en la retrospectiva del equipo. |
| **QA / DevOps Lead** | Garantizar la reproducibilidad del entorno, la calidad del código integrado y la tolerancia a fallos de todo el sistema (*la robustez técnica*). | • Asegurar que el despliegue con `docker compose up -d` funciona en limpio en cualquier equipo sin pasos manuales indocumentados.<br>• Definir la convención nomenclatura en Git.<br>• Diseñar scripts de pruebas automatizadas (`pytest` o tests de integración básicos de conexión contra MongoDB y Redis).<br>• Inyectar fallos deliberados para comprobar la resiliencia del worker (cortes de red, errores HTTP 500, datos nulos no contemplados). | • Archivos de infraestructura y configuración versionados (`docker-compose.yml`, `.env.example`, `.gitignore`).<br>• Enlaces a revisiones de código (*Code Reviews*) realizadas en los *Pull Requests* de sus compañeros.<br>• Script de pruebas de ejecución y reporte de incidencias críticas (*bugs*) detectadas antes de la entrega final |




### 2. Calificación y ponderación

La calificación final asignada al reto se compondrá de las siguientes partes:

- **Parte individual (70%)**
  - Examen técnico sobre el código (45%)
  - Memoria roles y evidencias (15%)
- **Parte grupal (30%)**
  - Coevaluación entre pares (10%) 
  - Producto final (repo, docker, demo) (30%)
``

La **nota mínima de corte** para promediar con la parte grupal es de al menos un **4,0 sobre 10** en el examen técnico individual.


### 3. Instrumentos de evaluación

#### A. Producto final y repositorio de equipo (30% - Nota Grupal)

Se evalúa el repositorio Git y la estabilidad del sistema durante la demo en vivo:

1. **Infraestructura (10%):** `docker-compose.yml` levanta en limpio sin errores; persistencia de volúmenes bien configurada.
2. **Pipelines de datos en batch (10%):** limpieza correcta en Pandas, inserción en MongoDB y precisión en las agregaciones analíticas solicitadas.
3. **Pipelines de datos en stream (10%):** polling fluido contra la API, encolado correcto por prioridad y vaciado/cierre de boxes hacia MongoDB ante la inyección de emergencias del profesor.

#### B. Examen técnico individual sobre el código (45% - Nota Individual)

Una prueba escrita en papel o en ordenador sin red (45 minutos en la Semana 7):

- **Parte 1: Dominio de su código propio (70% del examen):**
    - Ejemplos de preguntas: *"Justifica una función concreta que programaste (por qué usaste esa estructura de datos o cómo gestionaste los errores)"*, *"Indica cómo modificarías tu función `admitir_paciente()` para que admita un nuevo nivel Manchester 0 de máxima prioridad."*
- **Parte 2: Comprensión del sistema (30% del examen):**
    - Ejemplos de preguntas: *"Explica qué camino sigue un dato desde que entra por el polling de la API hasta que acaba en una colección de MongoDB, citando qué funciones de sus compañeros intervienen."*,  *"Si el box 3 se queda bloqueado en estado 'ocupado' pero la API ya envió el alta, ¿en qué archivo y qué línea del código de tu compañero buscarías el bug?"*


#### C. Memoria personal de roles y evidencias (15% - Nota Individual)

Un documento técnico breve (**máximo 2 páginas en PDF**) que cada alumno sube de forma individual. No debe ser una memoria teórica, sino un registro de evidencias:

**Periodo como Product Owner:**
- Enlaces a 2 ó 3 *User Stories* redactadas por tí, indicando cómo definiste los criterios de aceptación.
- Criterio de priorización aplicado para decidir qué tareas entraban en el sprint.

**Periodo como Scrum Master:**
- Captura del tablero Kanban al inicio y al final de tu periodo.
- Identificación de un bloqueo técnico que sufrió el equipo y cómo facilitaste su resolución.

**Aportación técnica individual al repositorio:**
- Enlaces a tus 3 *Pull Requests* o commits más significativos, indicando qué aportaciones realizaste en los mismos.



#### D. Coevaluación entre pares (10% - Ajuste Individual)

Al terminar la Semana 6, cada alumno cumplimenta de forma confidencial un formulario valorando a sus 2 compañeros de equipo sobre 4 dimensiones objetivas (escala 1 a 4):

| Dimensión                             | 1 (Insuficiente)                                                  | 2 (Mejorable)                                                   | 3 (Adecuado)                                            | 4 (Excelente)                                               |
| ------------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------- | ----------------------------------------------------------- |
| **Compromiso y puntualidad**          | Faltó a reuniones o entregó tarde sus partes bloqueando al resto. | Cumplió con retrasos que requirieron avisos del equipo.         | Entregó sus partes en las fechas pactadas por el grupo. | Siempre al día; proactivo adelantando trabajo.              |
| **Aportación técnica**                | Código deficiente que tuvo que ser reescrito por otros.           | Aportó código básico pero dependió en exceso de la ayuda ajena. | Resolvió de forma autónoma las tareas asignadas.        | Resolvió problemas complejos y ayudó a desbloquear a otros. |
| **Comunicación y transparencia**      | Desapareció días sin avisar del estado de sus tareas.             | Comunicación escasa; costaba saber si avanzaba.                 | Informó fluidamente en el tablero y canales de chat.    | Comunicación impecable; documentó y explicó sus cambios.    |
| **Facilitación en sus roles (PO/SM)** | Desatendió el tablero y no ejerció el rol asignado.               | Gestionó el rol de forma pasiva o solo cuando se le pidió.      | Mantuvo el backlog y las reuniones al día.              | Ejerció un liderazgo claro que dinamizó al equipo.          |

Para calcular la **nota de coevaluación** se toma la media de las puntuaciones recibidas de sus compañeros (máximo 16 puntos) y se traslada linealmente a la escala de 0 a 10. 

