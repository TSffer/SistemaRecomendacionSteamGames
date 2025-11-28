# Sistema de recomendacion Steam:

Se desarrolló un sistema de recomendación de videojuegos basado en contenido (*Content-Based Filtering*) usando el conjunto de datos de la plataforma Steam. Este motor utiliza **Embeddings** para comprender el contexto y temático de los juegos.

## El Dataset y Proceso de Enriquecimiento

### Fuente Original
Los datasets fueron obtenidos del repositorio de investigación de **Julian McAuley (UCSD)**:
🔗 [UCSD Steam Video Games Data](https://cseweb.ucsd.edu/~jmcauley/datasets.html)

1.  **australian_user_reviews.json**
    * Un dataset con 25799 reviews de usuarios.
2.  **steam_games.json**
    * Un dataset con 32135 juegos.

### Proceso de Enriquecimiento (Steam API)
La metadata original no contenia las descripciones de los videojuegos, por lo cual se implementó un notebook de extracción que consultó la **Steam Store API** (`store.steampowered.com/api/appdetails`) para cada `app_id`.

1.  **Fuente Base:** Se partió de un listado de *AppIDs* extraídos de las reseñas de usuarios (Australian User Reviews).
2.  **Extracción vía API:** Se desarrolló un script para consultar la **Steam Store API** (`store.steampowered.com/api/appdetails`).
3.  **Enriquecimiento:** Para cada juego, se extrajeron y limpiaron campos críticos que no existían en el set original:
    * `detailed_description`: La descripción narrativa completa del juego.
    * `short_description`: El "elevator pitch" del juego.
4.  **Limpieza:** Se eliminaron tags HTML y caracteres especiales.

*Resultado: Un dataset con 32135 juegos únicos con descripciones.*

## Diccionario de Datos

El proyecto se estructura en torno a dos conjuntos de datos principales procesados:

### 1. Dataset: `australian_user_reviews`
Contiene las interacciones explícitas de los usuarios.
* **`user_id`**: Identificador único del usuario.
* **`item_id`**: Identificador único del juego (Steam App ID).
* **`review_text`**: Texto libre con la opinión del usuario.
* **`recommend`**: Variable binaria (Target). `True` (1) si el usuario recomienda el juego, `False` (0) si no.

### 2. Dataset: `steam_games_enriched` (Contenido Enriquecido)
Información descriptiva utilizada para generar los vectores (Embeddings).
* **`publisher`**: Empresa que publicar el videojuego.
* **`genres`**: Lista de géneros oficiales (ej. *Action, RPG, Indie*).
* **`app_name`**: Nombre del videojuego.
* **`title`**: Título del videojuego.
* **`url`**: URL a la pagina del juego.
* **`release_date`**: Fecha de lanzamiento del juego.
* **`tags`**: Etiquetas generadas por usuarios.
* **`discount_price`**: Precio del juego con descuentos.
* **`reviews_url`**: URL donde se encuentran las reseñas.
* **`specs`**: Especificaciones técnicas (ej. *Single-player, Multi-player*).
* **`price`**: Precio base sin descuentos.
* **`early_access`**: Acceso anticipado
* **`id`**: Identificador único del juego en Steam (AppID).
* **`developer`**: Responsable del desarrollo del videojuego.
* **`sentiment`**: Clasificación del sentimiento general de reseñas en Steam
* **`metascore`**: Críticos profesionales.
* **`detailed_description`**: Descripción completa del juego proporcionada por Steam.
* **`short_description`**: Resumen comercial del juego.


## Arquitectura del Modelo

El sistema utiliza un enfoque vectorial para las recomendaciones:

* **Feature Engineering:** Creación de una "Concatenación de características" combinando *Título* (peso alto) + *Géneros* + *Tags* + *Descripción Detallada*.
* **Embeddings:** Utilización del modelo `text-embedding-3-small` de OpenAI para transformar el texto enriquecido en vectores de alta dimensionalidad.
* **Perfil de Usuario:** Construcción de un vector promedio ponderado basado únicamente en los juegos que el usuario ha recomendado explícitamente (Rating positivo = 1).
* **Similitud:** Cálculo de la Similitud del Coseno entre el vector del perfil de usuario y el inventario de juegos no jugados.

## Tecnologías Utilizadas

* **Google Colab**
* **Pandas & NumPy:** Manipulación de datos y operaciones vectoriales.
* **LangChain & OpenAI:** Generación de Embeddings.
* **Scikit-learn:** Cálculo de matrices de distancia (Linear Kernel).
* **Steam API:** Fuente de datos para el proceso de ETL.
* **Matplotlib & Seaborn:** Análisis exploratorio de datos (EDA).
