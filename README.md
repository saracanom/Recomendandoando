# Sistemas de Recomendación Híbridos
### Juan Felipe Vargas y Sara María Cano Martínez

Este repositorio contiene la implementación de dos enfoques de sistemas de recomendación sobre el dataset MovieLens:

1. **Filtrado Colaborativo** (ALS – Alternating Least Squares)  
2. **Filtrado Basado en Contenido** (TF-IDF)

> **Infraestructura**  
> - Ejecutado en AWS EMR (Elastic MapReduce) sobre un cluster Spark.
> - JupyterHub disponible en el nodo maestro para desarrollo interactivo.  
> - Los datos originales de MovieLens 32M fueron descargados de https://grouplens.org/datasets/movielens/32m/ y cargados en un bucket de S3.

---

## Estructura del Proyecto
### `data_files/`
- **movies_cleaned/**: resultados de Spark (shreds) de la base de datos `movies.csv` tras el procesamiento de los datos.
- **Origen de datos**: los CSV limpios se generaron a partir de los datos originales de MovieLens 32M cargados en un bucket S3. Los datos no se pueden subir a github por su gran tamaño.
- **ratings**: Los datos procesados de las calificaciones dadas por los usarios no se pueden subir debido a su tamaño pero se pueden obtener desde s3:
    ```
    aws s3 cp s3://recomendandoando/processed/clean/ratings/ratings_filtered.csv
    aws s3 cp s3://recomendandoando/processed/clean/movies/movies_filtered.csv
    aws s3 cp s3://recomendandoando/processed/ratings_filtered/part-00000-efea96bf-acf8-4725-94b8-ed8216e5dd68-c000.csv
    ```

### `data_processing/`
Notebooks para la **exploración** y **limpieza** de los datos originales:
- `Data_preparation.ipynb`: script en Python que aplica filtros, elimina valores duplicados y unifica formatos.  
- `Data_exploration.ipynb`: notebook con análisis exploratorio (visualizaciones, estadísticas básicas).

### `model_files/`
Implementaciones de los algoritmos de recomendación:
- `AWS_Content_Filter.ipynb`:  
  - Extracción de características de los ítems (p. ej. TF-IDF sobre géneros y títulos).  
  - Cálculo de similitud coseno y función de recomendación basada en contenido.
- `AWS_Colaborative_Filter.ipynb`:  
  - Entrenamiento de un modelo ALS con Spark MLlib.  
  - Funciones para calcular predicciones y métrica (RMSE).

### `saved_models/`
Modelos entrenados guardados mediante el mecanismo de Spark:
- `als_model/`: carpeta con _checkpoints_ y metadatos del modelo ALS.  
- `content_based_model/`: vectores y transformaciones serializadas para el sistema de contenido.
---
## Cómo usar

1. **Clonar el repositorio**  
   ```bash
   git clone https://github.com/tu_usuario/tu_repositorio.git
   cd tu_repositorio
  
2. **Configurar AWS**
    - Otorga permisos para EMR y S3 en tu cuenta de AWS. Puede ser mediante AWS CLI o a través de la consola web (AWS Management Console).
    - Asegúrate de que los datos de MovieLens 32M estén cargados en tu bucket S3 (s3://tu-bucket/movielens-32m/).
3. **Instalar dependencias**
      ```python
      pip install -r requirements.txt
4. **Preprocesar datos**
    - Ejecutar los Notebooks para generar los CSV limpios y los outputs en `data_files/`.
5. **Abrir Jupyter en el clúster**  
     1. Ve a la **consola de AWS EMR**.  
     2. Selecciona tu clúster.  
     3. En la pestaña **Aplicaciones**, haz clic en **Jupyter**.  
     4. Se abrirá la interfaz de JupyterHub en el nodo maestro, donde podrás ejecutar los notebooks de `model_files/`.
6. **Cargar y hacer predicciones**
    - ALS:
      ```PYTHON
      from pyspark.ml.recommendation import ALSModel
      als = ALSModel.load("s3://tu-bucket/saved_models/als_model")
    - Content based:
        ```PYTHON
        recs = spark.read.parquet("s3://recomendandoando/recommendations/content_based/parquet/")
        recs.show()
        
        # load pipeline and re-generate features if you ingest new movies
        pipeline = PipelineModel.load("s3://recomendandoando/models/tfidf_content_pipeline")
        new_movies = spark.read.csv("s3://...")         
        new_featurized = pipeline.transform(new_movies)
