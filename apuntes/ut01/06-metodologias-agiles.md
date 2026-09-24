```
------------- ESPECIALIZACIÓN EN INTELIGENCIA ARTIFICIAL Y BIG DATA -------------
---------------------------------------------------------------------------------

Módulo:                     SISTEMAS DE BIG DATA
Profesor:                   Víctor J. González
Unidad de Trabajo:          UT01. Introducción al Big Data
Apartado:                   6.- Metodologías ágiles
Resultados de aprendizaje:  ?
```

# 6. Metodologías ágiles: SCRUM

## 6.1. ¿Qué es Scrum y por qué lo usamos?

**Scrum** es un marco de trabajo ágil diseñado para abordar problemas complejos entregando soluciones de forma incremental e iterativa.

En proyectos de Big Data no sirve el modelo tradicional en cascada (diseñar todo en un papel durante meses y probarlo el último día). En sistemas distribuidos, almacenamiento masivo o pipelines de datos surgen errores imprevistos de red, cuellos de botella de memoria y problemas de esquemas desde el minuto uno.

Scrum nos obliga a:

- **Entregar valor funcional continuo:** al final de cada semana debe haber algo que compile, despliegue o procese datos reales (aunque sea un subconjunto mínimo).
- **Inspeccionar y adaptar:** si una tecnología, contenedor o consulta no rinde como esperábamos, cambiamos de enfoque en días, no en semanas.
- **Transparencia y corresponsabilidad:** todo el trabajo del equipo está visible en un tablero. Nadie trabaja aislado ni desconoce el estado del clúster.


## 6.2. Nuestro Ritmo: 1 Semana = 1 Sprint

Cada reto del módulo dura aproximadamente **6 semanas**, lo que equivale a **6 Sprints de 1 semana cada uno**.

Al disponer de **2 horas lectivas semanales**, la clase presencial no es para "empezar a pensar qué hacer", sino el punto de sincronización neurálgico del equipo. El desarrollo técnico se reparte entre la sesión presencial y el trabajo autónomo semanal.

### Estructura de la sesión semanal (100 minutos)

```
┌─────────────────┬─────────────────┬──────────────────┬────────────────────────┐
│  Sprint Review  │   Sprint Retro  │ Sprint Planning  │   Sprint Execution     │
│     (Demo)      │  (Mejora int.)  │ (Siguiente hito) │  (Desarrollo técnico)  │
│     20 min      │     10 min      │      10 min      │         80 min         │
└─────────────────┴─────────────────┴──────────────────┴────────────────────────┘

```

1. **Sprint Review (Demo técnica - 20 min):** el equipo demuestra en vivo al profesor el incremento funcional de la semana (servicios levantados, scripts ejecutados, datos transformados).
2. **Sprint Retrospective (10 min):** el equipo pone en común el avance de la semana: ¿funcionalidades implementadas? ¿qué cuello de botella tuvimos? ¿qué acuerdo tomamos para trabajar mejor esta semana?
3. **Sprint Planning (10 min):** selección y reparto de las historias/tareas técnicas del Product Backlog para la semana que empieza.
4. **Sprint Execution / Trabajo en aula (80 min):** configuración conjunta, desbloqueo de errores críticos con el profesor y arranque del trabajo técnico semanal.



## 6.3. Los roles del equipo

En este curso **todos los miembros son perfiles técnicos y todos programan/configuran**. Nadie asume un rol puramente burocrático. Los roles de gestión **rotarán**.

| Rol                    | Responsabilidad Ágil                                                                                                      | Responsabilidad Técnica en el Reto                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Product Owner (PO)** | Prioriza el Backlog, valida que el incremento cumpla los criterios de aceptación y se comunica con el profesor (cliente). | Valida esquemas de datos, define pipelines y comprueba que la salida de los datos sea correcta. |
| **Scrum Master (SM)**  | Vela por el cumplimiento de la metodología, gestiona el tablero Kanban y elimina bloqueos de equipo/entorno.              | Lidera la infraestructura, repositorios Git, scripts de despliegue (`docker-compose`) y ramas.  |
| **Data Engineer**      | El resto de miembros del equipo. Estiman y desarrolla tareas del Sprint Backlog.                                          | Focalizado en ingesta de fuentes, almacenamiento distribuido y conectores.                      |



## 6.4. Los artefactos de Scrum

Los artefactos representan el trabajo y el valor generado. Son el núcleo visible de vuestro proyecto.

### A. Product Backlog

Es la lista ordenada y viva de todo lo que necesita el reto para completarse con éxito. Lo gestiona principalmente el **Product Owner** y contiene **Historias de Usuario** o requerimientos técnicos desglosados (ej. *"Configurar clúster de 3 nodos de Cassandra con replicación 2"* o *"Diseñar script de ingesta concurrente en Python"*).

### B. Sprint Backlog

El subconjunto de tareas del Product Backlog que el equipo se compromete a completar **durante la semana en curso**.

- Cada tarea técnica debe poder completarse en un rango de **2 a 5 horas** de trabajo.
- Toda tarea tiene **un único responsable directo**, aunque reciba apoyo puntual.

### C. El tablero Kanban (GitHub Projects / GitLab / Trello)

Es el espejo del proyecto. Se organiza en 5 columnas obligatorias:

1. **Backlog:** tareas futuras del reto.
2. **Sprint To Do:** tareas comprometidas para la semana actual.
3. **In Progress:** tareas en las que se está trabajando activamente (máximo 1 tarea por persona a la vez).
4. **Review / Testing:** esperando validación cruzada o revisión de Pull Request por otro compañero.
5. **Done:** tarea finalizada bajo la *Definition of Done*.

### D. El incremento

La suma de todas las tareas completadas durante el sprint y los sprints anteriores. El incremento debe ser **utilizable y demostrable**. Una arquitectura que "compila en mi portátil pero no en el de los demás" no es un incremento.



## 5. Definition of Done (DoD) técnica

Una tarea técnica en Big Data no pasa a la columna **Done** solo porque alguien haya escrito el código. Para mover una tarea a **Done**, debe cumplir todos estos criterios:

```
[ ] 1. Código subido al repositorio.
[ ] 2. PR revisada y aprobada por al menos otro miembro del equipo.
[ ] 3. Infraestructura dockerizada y reproducible (levanta con un único comando).
[ ] 4. Sin credenciales ni contraseñas en plano dentro de los commits (uso de .env).
[ ] 5. README o documentación técnica breve actualizada con instrucciones de uso.
[ ] 6. Datos de prueba procesados con éxito y salida verificada.

```


## 6. ¿Qué hacemos en cada semana?

### 1. Sprint Planning (Al iniciar la clase)

El objetivo es definir el *Sprint Goal* (meta concreta de la semana) y seleccionar las tareas.

Algunas preguntas clave que nos podemos plantear son: ¿Qué entrega de valor técnico tendremos funcionando dentro de 7 días? ¿Quién asume cada tarea técnica?


### 2. Daily Standup (Sincronización semanal)

Aunque el marco formal lo hace a diario, en vuestro caso, al tener solo un día de clase a la semana, se realiza de dos formas:

* **Asíncrona (entre semana):** mediante un canal de comunicación del equipo (Teams, Discord, Telegram), cada miembro escribe a mitad de semana:
  1. *¿Qué he completado desde la clase?*
  2. *¿En qué estoy trabajando ahora?*
  3. *¿Tengo algún bloqueo técnico o de entorno?*
* **Presencial:** 5 minutos al entrar al aula antes de empezar a programar.

### 3. Sprint Review (La Demo Técnica)

* **Duración:** 5 minutos por equipo frente al profesor.
* **Formato:** Demostración en vivo en terminal, navegador o logs del clúster.
* **Prohibido:** Presentaciones de diapositivas explicando intenciones o código que no se ejecuta. Se evalúa el funcionamiento real.

### 4. Sprint Retrospective (La Inspección de Procesos)

- **Objetivo:** Mejorar la forma de trabajar para la semana siguiente.
- El equipo responde a tres preguntas rápidas:
  - **¿Qué hicimos bien esta semana?** (ej. *la partición de datos funcionó a la primera*).
  - **¿Qué no funcionó o nos retrasó?** (ej. *tardamos 3 días en darnos cuenta de que un puerto estaba cerrado en Docker*).
  - **¿Qué compromiso concreto aplicamos en el siguiente sprint?** (ej. *revisar los PRs en menos de 24 horas*).



## 7. Buenas prácticas

- **Trazabilidad en Git:** cada alumno debe reflejar su trabajo con commits descriptivos y atómicos desde su propio usuario. La actividad en Git es la prueba directa de la contribución individual.
- **Cero "Efecto Polizón":** en un equipo de 4 personas, si alguien no entrega su tarea semanal, el incremento se rompe y el bloqueo es evidente en la Review.
- **Pide ayuda rápido:** si llevas más de 1 hora bloqueado con un error de configuración de un contenedor o de red, comunícalo al Scrum Master y al equipo. La agilidad consiste en desbloquearse rápido, no en sufrir en silencio.