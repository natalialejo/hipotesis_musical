# Proyecto: hipotesis_musical

## Objetivo:
Explorar un conjunto de datos con el fin de identificar patrones o características que puedan determinar los factores que contribuyen al éxito de una canción teniendo como herramienta un extenso dataset de Spotify con información sobre las canciones más escuchadas en 2023.

## Contexto: 
Este proyecto se desarrolla en el marco de una discográfica que busca identificar los factores que hacen exitosa a una canción en plataformas de streaming. Con esta información,se pretende lanzar con éxito a un nuevo artista en el escenario musical global.

Las respuestas obtenidas nos ayudarán a realizar la validación de las siguientes hipótesis:

- **Hi1: Las canciones con un mayor BPM (Beats Por Minuto) tienen más éxito en términos de cantidad de streams en Spotify.**

- **Hi2: Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer.**

- **Hi3: La presencia de una canción en un mayor número de playlists se relaciona con un mayor número de streams.**

- **Hi4: Los artistas con un mayor número de canciones en Spotify tienen más streams.**

- **Hi5: Las características de la canción influyen en el éxito en términos de cantidad de streams en Spotify.**

## Metodología
### 1. Herramientas:
* Google Sheets
* BigQuery
* Power BI
* ChatGPT 
* Google Slides 
* Loom
  
### 2. Lenguajes:
* Lenguaje SQL en BigQuery.
* Lenguaje Python en Power BI.

### 3. Descripción de las variables del dataset:
Los datos se dividen en 3 tablas, la primera sobre el rendimiento de cada canción en Spotify, la segunda con el rendimiento en otras plataformas como Deezer o Apple Music, y la tercera con las características de estas canciones.

**track_in_spotify:**
- *track_id*: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite
- *track_name*: Nombre de la canción.
- *artist(s)_name*: Nombre del artista(s) de la canción.
- *artist_count*: Número de artistas que contribuyen a la canción.
- *released_year*: Año en que se lanzó la canción.
- *released_month*: Mes en el que se lanzó la canción.
- *released_day*: Día del mes en que se lanzó la canción.
- *in_spotify_playlists*: Número de listas de reproducción de Spotify en las que está incluida la canción.
- *in_spotify_charts*: Presencia y ranking de la canción en las listas de Spotify.
- *streams*: Número total de transmisiones en Spotify. Representa la cantidad de veces que la canción fue escuchada.

**track_in_competition:**
- *track_id*: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite
- *in_apple_playlists*: número de listas de reproducción de Apple Music en las que está incluida la canción
- *in_apple_charts*: Presencia y rango de la canción en las listas de Apple Music
- *in_deezer_playlists*: Número de listas de reproducción de Deezer en las que está incluida la canción
- *in_deezer_charts*: Presencia y rango de la canción en las listas de Deezer
- *in_shazam_charts*: Presencia y rango de la canción en las listas de Shazam

**track_technical_info:**
- *track_id*: Identificador único de la canción. Es un número entero de 7 dígitos que no se repite
- *bpm*: Pulsaciones por minuto, una medida del tiempo de la canción.
- *key*: Clave musical de la canción
- *mode*: Modo de la canción (mayor o menor)
- *danceability_%*: Porcentaje que indica qué tan adecuada es la canción para bailar
- *valence_*: Positividad del contenido musical de la canción.
- *energy_*: Nivel de energía percibido de la canción.
- *acusticness_*: Cantidad de sonido acústico en la canción.
- *instrumentality_*: Cantidad de contenido instrumental en la canción.
- *liveness_*: Presencia de elementos de actuación en vivo.
- *speechiness_*: Cantidad de palabras habladas en la canción.

## Procesamiento y preparación de datos:

- Creación del Proyecto y Conjunto de Datos en BigQuery:  
    * Proyecto: *proyecto2-hipotesis-426821*  
    * Tablas Importadas: *track_in_competition, track_in_spotify, track_technical_info*  

- Subida de tablas con nombres originales:
    * Se utilizó "Character Map V2" para aceptar el nombre *artist(s)_name*, cambiándolo a *artist_s_name*.
   
- Identificación de nulos y duplicados: 
    * Identificador Único: *track_id*
    * Nulos:Comandos SQL utilizados: `COUNT`, `WHERE`, `IS NULL`:
        En track_in_competition: 50 nulos en *shazam_charts*
        En track_in_spotify: No hay nulos
        En track_technicalinfo: 95 nulos en *key*
    * Duplicados: Comandos SQL utilizados: `COUNT`, `GROUP BY`, `HAVING`: 4 duplicados en *track_in_spotify*

- Manejo de nulos:
    * Se decidió mantener los nulos en *track_in_competition*.

- Eliminación de variables no utiles:  
    * Se eliminaron dos variables de *track_technical_info* y una de *track_in_spotify* utilizando `SELECT EXCEPT`.

- Manejo de datos discrepantes:  
    * Variables Categóricas: Uso de `LIKE` y `REGEXP` para manejar nombres con símbolos extraños.
    * Variables Numéricas: Uso de `MAX`, `MIN`, `AVG` para identificar valores discrepantes.

- Uso de `CAST` para modificar tipos de datos: `CAST` se utilizó en lugar de `UPDATE`para evitar modificar la tabla original y mantener una copia original del conjunto de datos.

- Creación de Variables Adicionales:  
    * Variables de *fecha_released*, *total_playlist* y *streams_int64* utilizando `CONCAT`, `CAST`, `LPAD` y `DATE`.


#### Ejemplo de consulta para variable *streams*

``` sql
SELECT
  streams,
  SAFE_CAST(IF(REGEXP_CONTAINS(streams, r'^[0-9]+$'), streams, NULL) AS INT64) AS streams_int64
FROM 
  `proyecto2-hipotesis-426821.Dataset_hipotesis.track_in_spotify`;
```
- Creación de tablas limpias y unión de estas: después de limpiar, modificar y crear vistas de las tres tablas, se utilizó la función `LEFT JOIN` para unirlas y crear la vista del dataset *consolidado_view*, con el fin de facilitar la actualización y manipulación de los datos sin alterar las tablas originales, permitiendo cambios dinámicos y eficientes una vez conectado a Power BI.

#### Ejemplo de consulta para dataset *consolidado_view*

``` sql
CREATE OR REPLACE VIEW `proyecto2-hipotesis-426821.Dataset_hipotesis.consolidado_view` AS
SELECT
S.track_id,
S.track_name,
S.artist_s__name,
S.released_day,
S.released_month,
S.released_year,
S.fecha_released,
S.in_spotify_charts,
S.in_spotify_playlists,
S.streams_int64,
C.in_apple_charts,
C.in_apple_playlists,
C.in_deezer_charts,
C.in_deezer_playlists,
C.in_shazam_charts,
T.bpm,
T.`acousticness_%`,
T.`danceability_%`,
T.`energy_%`,
T.`instrumentalness_%`,
T.`liveness_%`,
T.`speechiness_%`,
T.`valence_%`,
FROM `proyecto2-hipotesis-426821.Dataset_hipotesis.in_spotify_view` AS S
LEFT JOIN
`proyecto2-hipotesis-426821.Dataset_hipotesis.competition_view` AS C
ON
S.track_id = C.track_id
LEFT JOIN
 `proyecto2-hipotesis-426821.Dataset_hipotesis.tecnical_view` AS T
ON
S.track_id = T.track_id
ORDER BY
  S.artist_s__name,
  S.released_year,
  S.released_month,
  S.released_day;
```
## Análisis exploratorio:

- Debido a la limitación de Power BI en Mac, se siguieron estos pasos técnicos:
    * Instalación de Parallel Desktop: Para crear un ambiente de Windows en Mac.
    * Instalación de Power BI: Dentro del ambiente de Windows.
    * Conexión a BigQuery: Se conectaron las vistas *consolidado_view, in_spotify_view, competition_view* y *technical_view*.

- Una vez se tuvo el ambiente de trabajo listo, se procedió a la exploración y manipulación de datos:
    * De forma exploratoria, se crearon categorías  como *track_count* y *track _released_year* junto a gráficos correspondientes para una mejor comprensión.
    * Se calcularon medidas de tendencia central: `AVG`, `MAX`, `MIN`, `MEDIAN`, `STANDAR DEVIATION`: 

![Texto alternativo](img/graph_mtc.png?raw=true)

- Los resultados encontrados indican que hay una gran variabilidad en el número de streams y en el número de playlists en las que están las canciones. Aunque la mayoría de las canciones tienen un número relativamente bajo de streams y playlists, algunas pocas canciones alcanzan cifras extremadamente altas, lo que sesga los promedios hacia arriba. Los años de lanzamiento están mayormente concentrados en fechas recientes, y los BPM de las canciones muestran una variabilidad moderada.

- Además, se visualizó la distribución de los datos a través de histogramas, para ellos es necesario instalar el lenguaje python en Power BI y correr un script que devuelva dichos gráficos:

```python
import matplotlib.pyplot as plt
import pandas as pd

# The following code to create a dataframe and remove duplicated rows is always executed and acts as a preamble for your script:
dataset = dataset.drop_duplicates()

# Obtén los datos de PBI - solo necesita cambiar esta información de todo el código
data = dataset['streams_int64']

# Crea el histograma
plt.hist(data, bins=10, color='purple', alpha=0.7)
plt.xlabel('Valor')
plt.ylabel('Frecuencia')
plt.title('Histograma')

# Muestra el histograma
plt.show()
```

![Texto alternativo](img/graph_hist.png?raw=true)

- Se descubrió que en el *Histograma de Total de Streams*, la distribución de streams es asimétrica y sesgada a la derecha, con la mayoría de canciones acumulando una cantidad relativamente baja de streams y unas pocas canciones alcanzando un número extremadamente alto. La mayor frecuencia de canciones se encuentra en el rango más bajo de streams, cercano a 0, y disminuye rápidamente a medida que los streams aumentan. Los valores de streams varían desde casi 0 hasta 35 mil millones, siendo la mayoría de las canciones de menos de 1 mil millones de streams. De igual forma, la distrubución en el *Histograma de Total de Playlists* es similar a la de los streams, con un sesgo a la derecha donde la mayoría de las canciones están en un número bajo de playlists. La mayor frecuencia de canciones se encuentra en el rango más bajo, con menos de 10,000 playlists. A medida que aumenta el número de playlists, la frecuencia de canciones disminuye significativamente, con muy pocas canciones alcanzando más de 60,000 playlists.

- Para entender el comportamiento de los datos a lo largo del tiempo, se realizaron visualizaciones específicas:
  * Gráfico de Líneas de *track_id* by *released_year* para visualizar la cantidad de canciones lanzadas por año. Esto ayuda a identificar tendencias en la producción de música a lo largo del tiempo.
  * Gráfico de Líneas de *streams_int64* by *released_year* para analizar el total de streams por año, lo cual permite observar cómo ha evolucionado la popularidad de las canciones y el consumo de música en diferentes períodos.
 
#### Gráficos de línea
![Texto alternativo](img/graph_line.png?raw=true)

   

- Creación de Categorías por Cuartiles en BigQuery : Una vez realizada la exploración de datos, se procedió a crear categorías por cuartiles para las variables de características técnicas de las canciones utilizando consultas en BigQuery.

    * Generación de Cuartiles: se utilizó la función `NTILE` para dividir las variables en cuartiles, de esto se genera una tabla temporal.
    * Creación de Categorías: se creó una vista llamada categoria. Esta vista combina datos de la tabla *consolidado* con los cuartiles de la tabla *quartiles*, y además categoriza varias métricas (bpm, streams, danceability, valence, energy, acousticness, instrumentalness, liveness, speechiness) en categorías de "Bajo" (1,2 y 3 quartiles) y "Alto" (quartile 4).

#### Ejemplo de consulta para creación de vista de *categorías*

``` sql
CREATE OR REPLACE VIEW `proyecto2-hipotesis-426821.Dataset_hipotesis.categoria` AS
SELECT
  a.track_id,
  a.streams_int64,
  a.bpm,
  a.`acousticness_%`,
  a.`danceability_%`,
  a.`energy_%`,
  a.`instrumentalness_%`,
  a.`liveness_%`,
  a.`speechiness_%`,
  a.`valence_%`,
  q.q_bpm,
  q.q_streams,
  q.q_danceability,
  q.q_valence,
  q.q_energy,
  q.q_acousticness,
  q.q_instrumentalness,
  q.q_liveness,
  q.q_speechiness,
  IF(q.q_bpm IN (1, 2, 3), "Bajo", "Alto") AS bpm_category,
  IF(q.q_streams IN (1, 2, 3), "Bajo", "Alto") AS streams_category,
  IF(q.q_danceability IN (1, 2, 3), "Bajo", "Alto") AS danceability_category,
  IF(q.q_valence IN (1, 2, 3), "Bajo", "Alto") AS valence_category,
  IF(q.q_energy IN (1, 2, 3), "Bajo", "Alto") AS energy_category,
  IF(q.q_acousticness IN (1, 2, 3), "Bajo", "Alto") AS acousticness_category,
  IF(q.q_instrumentalness IN (1, 2, 3), "Bajo", "Alto") AS instrumentalness_category,
  IF(q.q_liveness IN (1, 2, 3), "Bajo", "Alto") AS liveness_category,
  IF(q.q_speechiness IN (1, 2, 3), "Bajo", "Alto") AS speechiness_category
FROM
  `proyecto2-hipotesis-426821.Dataset_hipotesis.consolidado` a
LEFT JOIN `proyecto2-hipotesis-426821.Dataset_hipotesis.quartiles` q
ON a.track_id = q.track_id
WHERE a.streams_int64 IS NOT NULL;
``` 
- **Observacion**:Al categorizar, se observaron asignaciones  que pueden tomarse como incorrectas debido a la alta concentración de valores cero en algunas variables. La función NTILE(4) distribuye los datos en cuartiles según su orden, pero en distribuciones sesgadas, con muchos ceros, no puede diferenciar entre los cuartiles. Esto sugiere que puede haber formas más adecuadas de manejar la categorización para reflejar mejor las características técnicas y su relación con el éxito.

## Análisis ténico:  

- Llegado a este punto, se analizaron las categorías (alto, bajo) creadas para las características de las canciones en relación con la variable *streams_int64*:
    * Se crearon tablas matrix en Power BI y se segmentaron y agruparon los datos. Se encontró que las diferencias en el promedio de streams entre las distintas categorías de características **no son significativamente grandes**,lo cual  sugiere que ninguna de estas características por sí sola tiene un impacto fuerte en el número de streams de una canción. Las variaciones encontradas son relativamente pequeñas, puede ser que existan otros factores que determinen el éxito de una canción en términos de streams.

- Cálculo de Correlaciones y validación de hipótesis:Se utilizó la función `CORR `en BigQuery para calcular la correlación entre las variables, para validar o refurtar las hipótesis planteadas:
    * Hipótesis 1: No se encontró relación significativa entre BPM y streams (CORR(bpm, streams_int64) = -0.002).
    * Hipótesis 2: Relación moderada a fuerte entre popularidad en Spotify y otras plataformas (CORR(in_spotify_charts, in_deezer_charts) = 0.60).
    * Hipótesis 3: Fuerte relación positiva entre número de playlists y streams (CORR(total_playlists, streams_int64) = 0.78).
    * Hipótesis 4: Fuerte relación positiva entre número de canciones de un artista y streams (CORR(in_spotify_playlists, streams_int64) = 0.79).
    * Hipótesis 5: No se encontró relación significativa entre características de la canción y streams.

- En Power BI, se visualizó la correlación utilizando scatter plots.

#### Ejemplo de consulta para la validación de hipotesis

``` sql
#Las canciones más populares en el ranking de Spotify también tienen un comportamiento similar en otras plataformas como Deezer. #

SELECT CORR (in_spotify_charts,in_deezer_charts) AS corr_hip2
FROM `proyecto2-hipotesis-426821.Dataset_hipotesis.consolidado_view` 

# 0.5999860553480  hipotesis validada #
```
#### Ejemplo de scatter plot para la visualización de hipotesis validada

![Texto alternativo](img/graph_hi3.png?raw=true)

#### Ejemplo de scatter plot para la visualización de hipotesis refutada  

![Texto alternativo](img/graph_hi5.png?raw=true)


