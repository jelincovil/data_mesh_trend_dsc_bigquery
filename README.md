# 📊 Data Trends Mesh en BigQuery

### Descripción  
Este proyecto consolida en **Google BigQuery** las principales **tendencias en ciencia de datos** (lenguajes, algoritmos, empresas, IA, países, proyectos, librerías) bajo un enfoque **Data Mesh**. Se estructura en dominios temáticos con capas **Bronze–Silver–Gold** para garantizar calidad y reutilización.

### Arquitectura  
- **Bronze:** datos crudos de APIs y CSV (StackOverflow, Kaggle, GitHub, PapersWithCode, HuggingFace, Glassdoor).  
- **Silver:** limpieza, unificación y normalización (ej. uso anual de lenguajes, popularidad de frameworks, conteo de ofertas de empleo).  
- **Gold:** tablas agregadas listas para análisis (Top 10 lenguajes por país, crecimiento de papers, top empresas contratantes, ranking de proyectos IA).

Cada dominio (Lenguajes, Algoritmos, Tecnologías, Publicaciones, Proyectos, Empresas) vive en su propio dataset de BigQuery, con tablas versionadas y metadatos. Se usa Python o R para extraer, transformar y cargar (ETL), y SQL/BigQuery para las consultas.

### Fuentes  
- **StackOverflow & Kaggle Surveys**: uso de lenguajes y frameworks.  
- **GitHub Archive & Trending API**: proyectos populares.  
- **PapersWithCode & OpenAlex**: publicaciones y métodos.  
- **HuggingFace Hub**: datasets y modelos.  
- **Glassdoor (Kaggle)**: ofertas de Data Scientist.

### Conexión local  
Desde Python/R se emplea el SDK de BigQuery:
```python
from google.cloud import bigquery
client = bigquery.Client(project="mi-proyecto")
df = client.query("SELECT * FROM dominio_gold WHERE year=2024").to_dataframe()
```
Desde R
```{r}
library(bigrquery)
df <- bq_project_query("mi-proyecto", "SELECT * FROM dominio_gold LIMIT 5") %>%
      bq_table_download()
```
---
## Ejemplo de implementación en Python


```python
from google.cloud import bigquery
from google.oauth2 import service_account
import pandas as pd

# 0) Instala los paquetes si aún no los tienes:
# 1) Carga tu clave de servicio y crea el cliente BigQuery
credentials = service_account.Credentials.from_service_account_file(
    "ds-trends-project-bc65a73a5bfb.json"
)
client = bigquery.Client(
    credentials=credentials,
    project="ds-trends-project",
    location="US"   # ajusta la región si tu dataset NO está en US
)

# 2) Lista los datasets disponibles en tu proyecto
print("Datasets en ds-trends-project:")
for ds in client.list_datasets():
    print(" •", ds.dataset_id)

# 3) Una vez veas el nombre correcto, asígnalo aquí:
dataset_id = "lenguajes"  # reemplaza con el nombre real de tu dataset

# 4) Define la tabla Bronze que quieres consultar
table_id = "languages_bronze_stackoverflow"  # o el nombre real de tu tabla Bronze

# 5) Construye y ejecuta la consulta SQL
query = f"""
SELECT *
FROM `ds-trends-project.{dataset_id}.{table_id}`
LIMIT 5
"""
job = client.query(query)
rows = job.result()  # espera a que termine

# 6) Crea un DataFrame manualmente (sin usar to_dataframe)
df = pd.DataFrame([dict(row) for row in rows])

# 7) Muestra las primeras filas
print(df.head())

```
