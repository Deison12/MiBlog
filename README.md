# Jekyll Cayman Starter (BAABD)

## Cómo publicar en GitHub Pages

1. Crea un repositorio en GitHub (p.ej. `mi-blog`).
2. Sube estos archivos a la raíz del repo.
3. Ve a **Settings → Pages → Build and deployment → Deploy from a branch**.
4. Selecciona la rama `main` y carpeta `/ (root)`. Guarda.
5. Espera a que termine el build y visita la URL que GitHub indique.

## Estructura

* `_config.yml` → configuración de Jekyll y tema Cayman.
* `index.md` → portada con lista de posts.
* `_posts/AAAA-MM-DD-analitica-clickstream.md` → tu post principal.
* `assets/` → coloca aquí `grafico_clicks.png`.

## Análisis Clickstream con PySpark y Google Colab

Este proyecto implementa un flujo completo de **simulación, procesamiento y análisis de datos de clickstream** utilizando **Python, PySpark, Pandas y Matplotlib** dentro de Google Colab.

El propósito es replicar cómo sistemas reales analizan actividad de usuarios, detectan patrones y generan visualizaciones para toma de decisiones.


###  Objetivo del análisis

* **Simular tráfico de usuarios** en una aplicación web
* Procesar eventos en **ventanas temporales de 1 minuto**
* Identificar **usuarios más activos**
* Visualizar resultados y exportarlos

Este tipo de análisis se utiliza en:

* Plataformas de streaming
* Apps móviles y web
* Sistemas de monitoreo de carga y ux
* Product analytics & growth

###  Flujo del proyecto

| Etapa                              | Descripción                                    |
| ---------------------------------- | ---------------------------------------------- |
| 1️⃣ Generación de datos sintéticos | Se crean 2,000 eventos → 30 usuarios → 2 horas |
| 2️⃣ Carga en PySpark               | Se lee el CSV y se convierte `timestamp`       |
| 3️⃣ Ventanas de tiempo             | Agrupación por usuario y ventana de 1 minuto   |
| 4️⃣ Ranking de usuarios            | Top 10 usuarios según número de clics          |
| 5️⃣ Visualización                  | Gráfico de barras en Matplotlib                |
| 6️⃣ Exportación                    | Descarga del gráfico desde Colab               |

###  Generación del dataset sintético

Se crean timestamps aleatorios, usuarios y clics, exportando a `clicks.csv`.

```python
import pandas as pd, numpy as np, datetime as dt

rng = np.random.default_rng(42)
n_rows = 2000
start = dt.datetime(2025, 10, 28, 20, 0, 0)
end   = dt.datetime(2025, 10, 28, 22, 0, 0)

timestamps = rng.uniform(start.timestamp(), end.timestamp(), size=n_rows)
timestamps = [dt.datetime.fromtimestamp(t).strftime("%Y-%m-%d %H:%M:%S") for t in timestamps]
users = [f"U{idx:03d}" for idx in range(1, 31)]
user_ids = rng.choice(users, size=n_rows, replace=True)
clicks = rng.integers(1, 4, size=n_rows)

pd.DataFrame({"Timestamp": timestamps, "User_ID": user_ids, "Clicks": clicks}) \
  .to_csv("clicks.csv", index=False)
```
###  Carga en PySpark y conversión del timestamp

```python
from pyspark.sql.functions import to_timestamp, col

schema = "Timestamp STRING, User_ID STRING, Clicks INT"
df = spark.read.csv("clicks.csv", header=True, schema=schema)

df = df.withColumn("ts", to_timestamp(col("Timestamp")))
df.show(5, truncate=False)
```
###  Ventanas temporales de 1 minuto

```python
from pyspark.sql.functions import window, sum as ssum

agg = (df.groupBy("User_ID", window(col("ts"), "1 minute"))
         .agg(ssum("Clicks").alias("clicks_min")))

agg.show(10, truncate=False)
```
###  Top 10 usuarios más activos

```python
from pyspark.sql.functions import desc, sum as ssum

top = (agg.groupBy("User_ID")
         .agg(ssum("clicks_min").alias("total_clicks"))
         .orderBy(desc("total_clicks"))
         .limit(10))

pdf = top.toPandas()
pdf
```
###  Visualización

```python
import matplotlib.pyplot as plt

plt.figure()
plt.bar(pdf['User_ID'], pdf['total_clicks'])
plt.title('Top 10 usuarios por clics (ventanas de 1 minuto)')
plt.xlabel('Usuario')
plt.ylabel('Total de clics')
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig('grafico_clicks.png', dpi=160)
plt.show()
```

### Descarga del gráfico

```python
from google.colab import files
files.download("grafico_clicks.png")
```

Guárdalo luego en la carpeta `assets/` del blog.

### Conclusiones

* Se simuló tráfico realista de interacción de usuarios
* PySpark permitió procesar y agrupar por ventanas temporales
* Identificamos patrones de actividad y usuarios más activos
* Matplotlib ayudó a visualizar resultados
* Flujo listo para analítica real y streaming

Fin del proceso ✅
