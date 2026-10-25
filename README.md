# Prueba Técnica – Data Engineer (SQL + PySpark)

Notebook: [Prueba_Técnica_DE_BP.ipynb](./Prueba_Técnica_DE_BP.ipynb)

Resolución de una prueba técnica para el rol de Data Engineer. Combina **SQL (SQLite)** y **PySpark**, y se ejecuta en Google Colab.

## Contenido

| Pregunta | Tecnología | Qué resuelve |
|---|---|---|
| 1 | SQL | Gol diferencia por equipo (goles a favor − en contra) a partir de la tabla `Partidos`, con grupo (A–F), puntos y un `GRAND TOTAL`. |
| 2 | SQL | 2.1: los 2 mejores equipos por grupo (desempate por gol diferencia). 2.2: rival, sede, ciudad, fecha y hora de la siguiente ronda (tabla `Clasificados`). |
| 3 | SQL | "Equipos Élite": los del quintil más alto de puntos acumulados en `Resultados_Qatar`. |
| 4 | PySpark | 4.1: DataFrames de jugadores y países. 4.2: marca del mejor jugador por país. 4.3: nuevo ranking con un indicador propio (goles por partido). 4.4: exporta `/tmp/Nuevo_Ranking.json`. |
| 5 | PySpark | 5.1: lectura del JSON de centros educativos de Madrid. 5.2: centro más céntrico por `centro_titularidad`. 5.3: exportación a Parquet, JSON y CSV. 5.4: justificación del formato. |

## Decisiones de diseño

**SQL**
- Los nombres de equipo en `Partidos` no son consistentes (tildes, puntos finales). Se normalizan para que cuadren con `Resultados_Qatar`.
- Se usan CTEs y funciones de ventana: `ROW_NUMBER` para el ranking por grupo y `COUNT(*) OVER()` para el quintil.
- En el quintil élite, los empates en el límite se incluyen.

**Pregunta 4**
- Indicador propio: `goles_por_partido = goles / partidos jugados`. Desempata por más goles y luego por el ranking anterior.
- El JSON usa la estructura del enunciado, incluida la clave `juadores` tal como aparece en el ejemplo.

**Pregunta 5**
- El archivo es un único arreglo JSON de varias líneas, por eso se lee con `multiLine=True`.
- Se excluyen los 4 centros sin coordenadas.
- La distancia euclidiana se calcula con una UDF: `sqrt((x2-x1)² + (y2-y1)²)`.
- Se elige el centro de menor distancia por grupo con `row_number`, desempatando por `centro_codigo`.

| Titularidad | Centro elegido |
|---|---|
| PRIVADO | 28062084 – INSTITUTO VOX |
| PRIVADO CONCERTADO | 28022323 – TEIDE II |
| PÚBLICO | 28018344 – SANTA TERESA DE JESUS |
| PÚBLICO-TITULARIDAD PRIVADA | 28060580 – ESCUELA DE FORMACION ESPEC. IMAGEN DIAG. HOSPITAL CLINICO SAN CARLOS |

**Pregunta 5.4: formato elegido**
- Se recomienda **Parquet** (idealmente como tabla Delta en Databricks). Es columnar, conserva el esquema, se comprime bien y es el formato nativo de Delta Lake.
- JSON y CSV se descartan para volúmenes grandes: son texto sin estadísticas ni compresión eficiente, y el CSV además no guarda esquema.

## Archivos

| Archivo | Descripción |
|---|---|
| `Prueba_Técnica_DE_BP.ipynb` | Notebook con todas las respuestas. |
| `test.db` | Base SQLite con `Partidos`, `Clasificados` y `Resultados_Qatar` (preguntas 1 a 3). |
| `CENTROS_EDUCATIVOS_MADRID.json` | Datos de la pregunta 5 (2638 centros). |

## Cómo ejecutarlo

**En Google Colab (recomendado)**
1. Sube `test.db` y `CENTROS_EDUCATIVOS_MADRID.json` a `/content`.
2. Ejecuta las celdas en orden. La celda 5.3 reutiliza el DataFrame `df_mas_centrico` de la 5.2.

**En local**
- Requiere Python con `pyspark` y Java 17.
- Con JDK 25, Spark 4.x falla.
- En Windows, define `PYSPARK_PYTHON` con el ejecutable de Python en uso para evitar el error *"Timed out while waiting for the Python worker"*.
- Las rutas `/content` y `/tmp` están pensadas para Colab; ajústalas si ejecutas en local.

## Notas

- El enunciado menciona `CENTROS_EDUCACION_MADRID.json`, pero el archivo real se llama `CENTROS_EDUCATIVOS_MADRID.json`.
- Los nombres de país del catálogo de la pregunta 4 traen un espacio no separable (U+00A0), que se limpia antes de unir los DataFrames.
