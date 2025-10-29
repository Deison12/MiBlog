---
layout: default
title: "Análisis de Flujo de Datos Simulado con Spark"
---

{% include nav.html %}
<link rel="stylesheet" href="{{ site.baseurl }}/assets/style.css">

# 📊 Análisis de Flujo de Clickstream con PySpark

Este proyecto tiene como objetivo procesar un flujo simulado de clics para identificar patrones de navegación de usuarios utilizando **PySpark** y técnicas de **ventanas temporales**. Es parte del ejercicio académico de *Big Data & Analytics*, donde se busca comprender cómo Spark procesa series de tiempo y aplica agregaciones por intervalos.

---

## 🎯 Objetivo del análisis

- Simular un dataset de **clickstream**  
- Procesarlo con **PySpark**  
- Aplicar **ventanas de 1 minuto**  
- Obtener el **Top 10** de usuarios con más clics  
- Visualizar resultados y **analizar patrones**

---

## 📂 Datos utilizados

El dataset contiene 2000 eventos de clics con las columnas:

| Campo | Descripción |
|---|---|
| `Timestamp` | Fecha y hora del clic |
| `User_ID` | Identificador del usuario (U001…U030) |
| `Clicks` | Cantidad de clics registrados |

Si el sistema no proporciona datos, estos se pueden **simular** (como hicimos aquí).

---

## ⚙️ Entorno de trabajo

- **Google Colab**  
- **PySpark 3.5.1**  
- **Python 3.x**  
- **Matplotlib** para graficar

### Instalación y sesión Spark

{% raw %}
```python
!pip install pyspark==3.5.1

from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Clickstream").getOrCreate()
spark  # devuelve info de la sesión
{% endraw %}
{% raw %}
import pandas as pd, numpy as np, datetime as dt

# Dataset sintético: 2000 eventos entre 20:00 y 22:00 para 30 usuarios
rng = np.random.default_rng(42)
n_rows = 2000
start = dt.datetime(2025, 10, 28, 20, 0, 0)
end   = dt.datetime(2025, 10, 28, 22, 0, 0)

timestamps = rng.uniform(start.timestamp(), end.timestamp(), size=n_rows)
timestamps = [dt.datetime.fromtimestamp(t).strftime("%Y-%m-%d %H:%M:%S") for t in timestamps]
users = [f"U{idx:03d}" for idx in range(1, 31)]
user_ids = rng.choice(users, size=n_rows, replace=True)
clicks = rng.integers(1, 4, size=n_rows)  # 1..3 clics

pd.DataFrame({"Timestamp": timestamps, "User_ID": user_ids, "Clicks": clicks}) \
  .to_csv("clicks.csv", index=False)

# Vista rápida
pd.read_csv("clicks.csv").head()
{% endraw %}
{% raw %}
from pyspark.sql.functions import to_timestamp, col

schema = "Timestamp STRING, User_ID STRING, Clicks INT"
df = spark.read.csv("clicks.csv", header=True, schema=schema)

df = df.withColumn("ts", to_timestamp(col("Timestamp")))

df.printSchema()
df.show(5, truncate=False)
{% endraw %}
{% raw %}
from pyspark.sql.functions import window, sum as ssum

agg = (df.groupBy("User_ID", window(col("ts"), "1 minute"))
         .agg(ssum("Clicks").alias("clicks_min")))

agg.show(10, truncate=False)
{% endraw %}
{% raw %}
from pyspark.sql.functions import desc, sum as ssum

top = (agg.groupBy("User_ID")
         .agg(ssum("clicks_min").alias("total_clicks"))
         .orderBy(desc("total_clicks"))
         .limit(10))

top.show()

# A Pandas para graficar
pdf = top.toPandas()
pdf
{% endraw %}
{% raw %}
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
{% endraw %}

{% raw %}
from google.colab import files
files.download("grafico_clicks.png")
{% endraw %}

<div style="text-align:center;">
  <img src="{{ site.baseurl }}/assets/grafico_clicks.png" alt="Gráfico de clics por usuario" width="80%">
</div>
