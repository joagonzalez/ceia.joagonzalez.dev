# Guión — Defensa Final CEIA FIUBA
**Optimización de intervenciones automatizadas contra discursos de odio en redes sociales**  
Mg. Ing. Joaquín González · Abril 2026

> Duración estimada: 25–30 minutos (incluye ~5 min de videos)  
> Ritmo aproximado: 1 minuto por slide, salvo indicación contraria

---

## Slide 1 — Título *(~30 seg)*

Buenas tardes. Mi nombre es Joaquín González y hoy presento mi trabajo final de la Carrera de Especialización en Inteligencia Artificial de FIUBA, titulado *Optimización de intervenciones automatizadas contra discursos de odio en redes sociales*.

Quiero agradecer al jurado y al director del trabajo, el Dr. Juan Manuel Ortiz de Zarate, por el tiempo y la dedicación.

---

## Slide 2 — El problema *(~1 min)*

En X.com se publican millones de posteos por día. El discurso tóxico y polarizante se propaga dentro de ese volumen, y la moderación humana es estructuralmente insuficiente para hacerle frente a esa escala.

El problema no es solo la detección: es la respuesta. Cuando un contenido tóxico alcanza miles de impresiones sin ningún tipo de contranarración, el daño ya está hecho. La pregunta que motiva este trabajo es: ¿puede la IA actuar como contrapeso, a escala, de forma automatizada?

---

## Slide 3 — Normsy *(~1.5 min)*

Normsy es la plataforma donde eso ocurre. Es una iniciativa no partidaria desarrollada por Data Voices para Civic Health Project, en producción desde enero de 2023. Detecta posteos tóxicos, genera una contranarrativa con un LLM, la publica como respuesta en X.com y recolecta métricas de engagement.

Este trabajo abarca tres contribuciones concretas sobre esa plataforma:  
primero, una plataforma analítica de datos que antes no existía;  
segundo, un framework para evaluar clasificadores de toxicidad de forma sistemática;  
y tercero, el fine-tuning del clasificador binario para mejorar la precisión.

La línea de tiempo muestra el recorrido: Normsy arrancó con operadores humanos en 2023, en 2024 sumamos el pipeline de datos y el framework, en Q3 2025 desplegamos las cuentas automatizadas, y en el período 2025–2026 trabajamos el fine-tuning y la evaluación que hoy presento.

---

## Slide 4 — Video: Normsy en acción *(~2 min — reproducir video)*

> ▶ Reproducir `normsy_intro.mp4`

Antes de entrar en los detalles técnicos, quiero mostrar brevemente cómo funciona Normsy en la práctica: desde que se detecta un tweet tóxico hasta que se publica la intervención.

---

## Slide 5 — Flujo de intervención *(~1 min)*

Este diagrama muestra el flujo completo de una intervención. Un post es detectado por el clasificador de toxicidad, pasa por el módulo de generación de respuesta usando un LLM, y la intervención es publicada en X.com a través de la API de Twitter. Posteriormente se recolectan métricas de engagement: impresiones, likes, respuestas recibidas.

Este es el corazón operacional del sistema. Todo lo que voy a presentar a continuación —los pipelines de datos, el framework de evaluación, el fine-tuning— gira alrededor de hacer este flujo más preciso y escalable.

---

## Slide 6 — Solución de datos *(~1.5 min)*

Cuando empecé a trabajar sobre los datos de Normsy, la situación era esta: toda la información vivía en MongoDB Atlas, más de 800.000 registros, sin posibilidad de hacer joins ni análisis exploratorio sistemático. No había dashboards, no había EDA histórico.

La decisión de diseño fue mover los datos a PostgreSQL en un servidor Hetzner, aplicando el patrón CQRS: lectura y escritura separadas. MongoDB sigue siendo la fuente transaccional —la plataforma escribe ahí sin impacto— y PostgreSQL es el destino analítico sobre el que corren Grafana y Jupyter.

Implementé esto con dos estrategias complementarias que vemos en el próximo slide.

---

## Slide 7 — Dos estrategias de procesamiento *(~1.5 min)*

Para mantener PostgreSQL sincronizado con MongoDB implementé dos mecanismos en paralelo.

El primero es un pipeline batch en Apache Airflow: el `smd_multicollection_etl` que ven en el diagrama. Extrae datos de MongoDB en lotes, los transforma aplicando funciones específicas por colección, los carga a PostgreSQL con truncate y reemplazo, y al final guarda métricas de ejecución y limpia los archivos temporales. Cubre las 17 colecciones principales y tiene recovery automático ante fallos.

El segundo es CDC en tiempo real, basado en Kafka, que detallo en el próximo slide.

---

## Slide 8 — Change Data Capture *(~1 min)*

El pipeline CDC captura los cambios en MongoDB en menos de un segundo. Debezium lee el oplog de MongoDB Atlas, publica los eventos en Kafka, y un consumer Python los persiste en PostgreSQL. Esto permite tener los datos casi en tiempo real sin impactar la base transaccional.

La combinación batch + CDC cubre dos necesidades distintas: el batch hace resincronizaciones completas cuando necesitamos consistencia total; el CDC mantiene el espejo actualizado en el día a día.

---

## Slide 9 — Análisis exploratorio: cuentas automatizadas *(~2 min)*

Con la plataforma analítica funcionando, pude hacer por primera vez un análisis histórico completo de los datos de Normsy. El hallazgo más relevante: de las 111.000 intervenciones totales, el 78% fue publicado por cuentas automatizadas, que se desplegaron en Q3 2025.

Lo importante no es solo el volumen: la mediana de impresiones por intervención es 8 para cuentas humanas y 5 para automatizadas —diferencia pequeña. Esto confirma que el escalado automatizado no degrada la calidad individual de las intervenciones.

*[avanzar fragmento]* La distribución de impresiones es comparable entre ambos tipos de cuenta.

*[avanzar fragmento]* Y las intervenciones generan interacción real: hay respuestas, lo que indica que el contenido llega a su audiencia y genera conversación.

---

## Slide 10 — El problema del clasificador *(~1 min)*

El análisis de datos también reveló un problema concreto: la precisión del clasificador baseline en producción es 0,396. Eso significa que casi 1 de cada 2,5 intervenciones se activa sobre contenido que no es tóxico.

El costo de un falso positivo en este contexto es alto: se publica una respuesta innecesaria en X.com, se generan fricciones públicas y se erosiona la credibilidad de la plataforma. No es solo un número estadístico —tiene impacto operacional y reputacional.

Esto motiva la segunda y tercera parte del trabajo.

---

## Slide 11 — Framework de evaluación *(~1.5 min)*

Para atacar ese problema de forma sistemática, construí un framework de evaluación de clasificadores. Evalúa 13 modelos de 5 proveedores —OpenAI, Google, Anthropic, xAI y Alibaba— en tres escalas distintas, midiendo F1, precisión, recall, costo en dólares y latencia.

El dataset golden tiene 400 posts balanceados: 100 por clase, ninguno expuesto al entrenamiento. Un YAML define el experimento completo, el CLI lo ejecuta automáticamente sobre cada combinación modelo × prompt, y los resultados se persisten en PostgreSQL.

El diseño es reproducible por construcción: el mismo YAML siempre produce los mismos resultados, versionados en Git.

---

## Slide 12 — Framework: configuración YAML *(~1 min)*

Este es el aspecto concreto de un experimento: un archivo YAML define el dataset, la lista de modelos, los prompts y la escala. El CLI recorre cada combinación automáticamente.

Para este trabajo ejecuté 13 modelos × 3 prompts = 39 evaluaciones en un solo experimento. El workflow es simple: el CLI lee el YAML, carga el dataset golden, clasifica cada post con cada modelo, calcula las métricas y las persiste con timestamp.

---

## Slide 13 — Video: Plataforma en acción *(~2 min — reproducir video)*

> ▶ Reproducir video de YouTube

Veamos la plataforma analítica en funcionamiento: los dashboards de Grafana, el pipeline ETL corriendo en Airflow, y las métricas en tiempo real.

---

## Slide 14 — Escala 0–10 vs 0–3 *(~1.5 min)*

Uno de los primeros experimentos con el framework fue comparar escalas de clasificación. La escala 0–10 que estaba en producción tiene un problema estructural: alta ambigüedad entre niveles intermedios, ruido subjetivo en la anotación, y alta dispersión para los modelos.

Al colapsar a escala 0–3, la tarea se vuelve más clara y los modelos responden mejor. El resultado es una mejora de 16,6 puntos en F1 en todos los modelos evaluados, de forma consistente. Esta fue la primera decisión concreta adoptada en producción.

---

## Slide 15 — Frontera de Pareto *(~1.5 min)*

Con los resultados del framework, apliqué análisis de frontera de Pareto para identificar los mejores modelos en el trade-off costo versus F1.

Los modelos sobre la frontera son grok-4-fast, gemini-2.0-flash, gemini-2.5-flash-lite y qwen-flash. Un hallazgo importante: ninguna configuración con escala 0–10 alcanza la frontera, lo que confirma la superioridad de la escala 0–3 también desde el ángulo de la eficiencia económica.

Esto orientó la selección del modelo base para el fine-tuning.

---

## Slide 16 — Precisión baseline *(~1 min)*

Pero incluso con la escala optimizada, la precisión de todos los modelos base sigue siendo baja. El gráfico muestra que en clasificación binaria con escala 0–3, ningún modelo base supera 0,5 de precisión de forma consistente.

El framework hizo visible un problema que antes no era medible: sin él, sabíamos que había falsos positivos, pero no teníamos forma de cuantificarlos ni de comparar alternativas sistemáticamente.

---

## Slide 17 — Hipótesis fine-tuning *(~30 seg)*

Esto lleva a la pregunta central de la tercera parte del trabajo: ¿puede el fine-tuning reducir los falsos positivos y mejorar la precisión de forma sostenida?

La hipótesis es que ajustar un modelo con datos del dominio específico —posteos políticos en inglés— puede orientarlo a ser más conservador y preciso.

---

## Slide 18 — Fine-tuning en Vertex AI *(~1.5 min)*

Entrené cuatro versiones de modelos Gemini usando Supervised Fine-Tuning con la técnica LoRA en Vertex AI. Las versiones 1 a 3 usan escala 0–3 con distintos modelos base: gemini-2.5-flash, gemini-2.5-flash-lite, y gemini-2.0-flash. La versión 4 usa tarea directamente binaria.

El dataset de entrenamiento tiene 1.300 ejemplos. Una observación relevante: la versión 4 —tarea binaria directa— no logra superar al modelo base. Esto sugiere que 1.300 ejemplos no son suficientes cuando el modelo base ya tiene conocimiento previo fuerte sobre la tarea. Escalar el dataset es el paso natural.

---

## Slide 19 — SFT vs Baseline *(~1.5 min)*

Los resultados del fine-tuning son claros: los modelos SFT superan consistentemente al baseline de producción en precisión. El mejor resultado es SFT v3, gemini-2.0-flash, con precisión 0,649 —un incremento del 64% respecto al baseline de 0,396.

Hay un trade-off: la mejora en precisión implica reducción en recall. Pero dado que en este dominio el costo de intervenir sobre contenido no tóxico es mayor que el de omitir uno tóxico, ese equilibrio es aceptable.

---

## Slide 20 — Tabla resumen *(~2 min)*

Esta tabla consolida los resultados de todo el trabajo. Partimos de un baseline 0–10 con precisión 0,409. Al pasar a escala 0–3 sin fine-tuning llegamos a 0,465. Con SFT v3 alcanzamos 0,649. Y el hallazgo más inesperado está en la última fila: un prompt binario simple, sin ningún entrenamiento adicional, llega a precisión 0,654 y F1 0,714 —los mejores resultados globales.

El camino recorrido en términos de precisión es claro: de 0,396 a 0,649 con SFT, y hasta 0,668 con el enfoque binario. Una mejora del 68,9% sobre el punto de partida.

---

## Slide 21 — Hallazgos *(~1 min)*

Tres hallazgos que considero los más relevantes del trabajo.

Primero, los modelos base ya contienen conocimiento significativo sobre toxicidad en el dominio político-partidario —el fine-tuning mejora, pero no transforma radicalmente.

Segundo, 1.300 ejemplos resultan insuficientes para superar ese conocimiento previo en una tarea directamente binaria. El camino es escalar el dataset.

Tercero, y quizás el hallazgo más práctico: el prompt binario es una alternativa de bajo costo operacional con rendimiento comparable al SFT actual. Esto tiene implicaciones directas para producción.

---

## Slide 22 — Impacto en producción *(~1 min)*

Los resultados del trabajo se tradujeron en decisiones concretas adoptadas por el equipo de Civic Health Project.

Primero: migración de la escala de clasificación 0–10 a 0–3 en producción.  
Segundo: adopción de la familia Gemini como clasificador de producción.  
Tercero: despliegue de las cuentas automatizadas, que llevaron el total de intervenciones a más de 111.000.

Este trabajo también representa la primera explotación sistemática de los datos históricos de Normsy desde su puesta en producción en 2023. Antes no había infraestructura para hacerlo.

---

## Slide 23 — Conclusiones y próximos pasos *(~2 min)*

En síntesis, el trabajo entrega tres contribuciones que se retroalimentan:

El pipeline analítico habilita por primera vez el análisis histórico de Normsy. Las cuentas automatizadas demuestran que el escalado no degrada la calidad. El cambio de escala mejora todos los modelos. El SFT mejora la precisión en un 64%. Y el prompt binario emerge como una alternativa competitiva sin costo de entrenamiento.

Sobre los próximos pasos: el más inmediato es escalar el dataset de ajuste para que el SFT pueda converger en tareas binarias. A continuación, explorar DPO —Direct Preference Optimization— para tener un control más fino del balance precisión-recall. Y a más largo plazo, explorar embeddings multimodales con clasificadores livianos como SVM o XGBoost, que eliminen el costo de inferencia por post.

---

## Slide 24 — Cierre *(~30 seg)*

Muchas gracias. Fue un trabajo de más de dos años sobre un sistema real, con datos reales y decisiones que se implementaron en producción. Espero que haya sido claro en la exposición.

Quedo a disposición para las preguntas del jurado.

---

*Tiempo total estimado sin videos: ~22 min | Con videos (~5 min): ~27 min*
