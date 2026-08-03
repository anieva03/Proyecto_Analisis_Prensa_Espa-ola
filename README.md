# Análisis de la Opinión Española

## Descripción General

**Análisis de la Opinión Española** es un proyecto de Procesado del Lenguaje Natural (NLP) que estudia cómo distintos medios de comunicación digitales españoles, aun cubriendo los mismos hechos noticiosos, lo hacen desde marcos ideológicos diferenciados que se reflejan en el lenguaje, los temas priorizados y el tono adoptado.

El proyecto construye un corpus propio de artículos de opinión de cinco periódicos digitales (La Razón, El Español, El Diario, Público y Tercera Información) y aplica un pipeline completo de NLP: scrapeo, limpieza, análisis exploratorio, topic modeling, análisis de polaridad, clasificadores supervisados y, como fase final, un sistema de agentes que recomienda noticias personalizadas.

---

## Características Principales

* Scrapeo automatizado de la sección de opinión de 5 periódicos digitales
* Limpieza y normalización lingüística del corpus (stopwords, puntuación, lematización)
* Análisis exploratorio mediante wordclouds por periódico
* Topic Modeling con embeddings y BERTopic
* Análisis de polaridad por tema y centrado en la figura de Pedro Sánchez
* Clasificadores supervisados (Random Forest y XGBoost) para predecir periódico, ideología y tema
* Sistema de agentes basado en LLM (LangChain + Llama 3.3 70B) para recomendación personalizada de noticias
* Interfaces en Streamlit para usuarios y para monitorización del orquestador
* Arquitectura de despliegue serverless con automatización diaria (cron job)

---

## Arquitectura del Sistema

El proyecto se divide en los siguientes bloques:

### Bloque 1 — Scrapeo

* Recolección de artículos de opinión de La Razón, El Español, El Diario, Público y Tercera Información
* Arquitectura en dos fases: recolección de URLs por paginación/sitemaps y visita individual a cada artículo
* Uso de `requests`, `BeautifulSoup`, `Selenium` (para webs con contenido dinámico o antibot), `re` y `urllib.parse`
* Manejo de problemas específicos por periódico: duplicados, rate limiting, paywalls, banners de cookies, detección antibot, paginación poco fiable

### Bloque 2 — Limpieza de los datos

* Eliminación de noticias duplicadas y de datos faltantes
* Reducción a 2000 artículos por periódico y unificación en un único dataframe
* Limpieza lingüística: eliminación de stopwords (conservando las negativas), signos de puntuación, números y caracteres extraños, y lematización

### Bloque 3 — Análisis Exploratorio de Datos (EDA)

* Generación de wordclouds por periódico para observar diferencias temáticas entre medios de derecha e izquierda

### Bloque 4 — Análisis en profundidad

* **Topic modeling**: embeddings con `paraphrase-multilingual-MiniLM-L12-v2` + `BERTopic` (UMAP + HDBSCAN + c-TF-IDF)
* **Análisis de polaridad por tema**: modelo `TextBlob` sobre el corpus limpio traducido al inglés
* **Análisis de polaridad sobre Pedro Sánchez**: modelo RoBERTa en español vía `pysentimiento`, aplicado al texto original sin limpieza lingüística
* **Clasificadores supervisados**: Random Forest y XGBoost para predecir periódico, ideología y tema a partir de embeddings reducidos por UMAP

### Bloque 5 — Sistema de agentes

* Agente basado en LangChain y Llama 3.3 70B Instruct (vía OpenRouter) que actúa como curador automático de noticias
* Herramientas del agente: `leer_texto_noticia` (consulta el cuerpo completo si el titular parece relevante) y `enviar_email` (envío del boletín HTML final)
* Perfiles de usuario e histórico de noticias almacenados en Google Sheets
* Interfaces en Streamlit: panel de preferencias de usuario (con chat sobre el histórico de noticias) y panel interno de monitorización del orquestador
* Arquitectura de despliegue serverless: repositorio y CI/CD en GitHub, servidor Flask en Render, envío de correo vía API de Brevo (evitando restricciones SMTP) y ejecución asíncrona programada mediante cron job diario

---

## Estructura del Proyecto

```
NLP_trabajo/
│
├── scrapers/
│   ├── el_espanol.py
│   ├── tercera_informacion.py
│   ├── la_razon.py
│   ├── publico.py
│   └── el_diario.py
│
├── limpieza.py
├── eda_wordclouds.py
├── topic_modeling.py
├── analisis_polaridad.py
├── clasificadores.py
│
├── agente/
│   ├── orquestador.py
│   ├── tools.py
│   └── requirements.txt
│
├── interfaces/
│   ├── panel_usuario.py
│   └── panel_monitorizacion.py
│
└── README.md
```

---

## Instalación

1. Clona el repositorio:

```bash
git clone https://github.com/avaljuan/NLP_trabajo.git
cd NLP_trabajo
```

2. Instala las dependencias:

```bash
pip install -r requirements.txt
```

3. Configura las variables de entorno necesarias:

* Credenciales de acceso a Google Sheets
* API Key de OpenRouter (para el modelo Llama 3.3 70B Instruct)
* API Key de Brevo (envío de correo)
* Contraseña/remitente del email del agente

---

## Uso

### Ejecución por etapas

```bash
python scrapers/el_espanol.py
python scrapers/tercera_informacion.py
python scrapers/la_razon.py
python scrapers/publico.py
python scrapers/el_diario.py

python limpieza.py
python eda_wordclouds.py
python topic_modeling.py
python analisis_polaridad.py
python clasificadores.py
```

### Sistema de agentes

```bash
python agente/orquestador.py
```

El orquestador ejecuta los scrapers, unifica y actualiza el histórico en Google Sheets, y lanza el agente LLM para generar y enviar los boletines personalizados a cada usuario registrado.

### Interfaces web

```bash
streamlit run interfaces/panel_usuario.py
streamlit run interfaces/panel_monitorizacion.py
```

* El **panel de usuario** permite crear un perfil (nombre, email, contraseña hasheada), configurar intereses informativos, decidir si se recibe el boletín diario y consultar el histórico de noticias mediante un chat.
* El **panel de monitorización** permite lanzar el orquestador y ver en tiempo real las métricas de ejecución (fuentes consultadas, noticias recopiladas, noticias nuevas, usuarios cargados, emails enviados).

**Nota:** En producción, el agente se ejecuta automáticamente cada día mediante una petición GET programada a la ruta `/disparar-agente` del servidor Flask desplegado en Render.

---

## Tecnologías Utilizadas

* Python
* `requests`, `BeautifulSoup (bs4)`, `Selenium` (scrapeo)
* `re`, `urllib.parse` (limpieza y validación de URLs)
* `paraphrase-multilingual-MiniLM-L12-v2`, `BERTopic` (UMAP + HDBSCAN + c-TF-IDF) para topic modeling
* `TextBlob` y `pysentimiento` (RoBERTa en español) para análisis de polaridad
* `Random Forest`, `XGBoost` (clasificación supervisada)
* `LangChain` + Llama 3.3 70B Instruct (vía OpenRouter) para el agente de recomendación
* `Streamlit` (interfaces de usuario y monitorización)
* Google Sheets (almacenamiento de perfiles e histórico de noticias)
* Flask + Render (despliegue serverless), Brevo API (envío de email), Cron Job (automatización diaria)

---

## Posibles Mejoras

* Utilizar modelos de polaridad nativos en español en todas las fases del análisis (en lugar de TextBlob con traducción al inglés)
* Ampliar el corpus con más periódicos y mayor volumen de artículos por cabecera
* Incorporar variables adicionales (estilo, sintaxis) para mejorar la predicción del periódico de origen
* Mejorar el balanceo de clases en la predicción de ideología
* Extender el sistema de agentes con más fuentes de personalización y canales de entrega
