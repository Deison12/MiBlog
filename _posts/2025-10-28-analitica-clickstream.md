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
```
{% endraw %}

### Generación de un Dataset Sintético de Eventos Clickstream para 30 Usuarios
{% raw %}
```python

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
```
{% endraw %}
### Carga del Dataset Clickstream en PySpark y Conversión del Timestamp
{% raw %}
```python
from pyspark.sql.functions import to_timestamp, col

schema = "Timestamp STRING, User_ID STRING, Clicks INT"
df = spark.read.csv("clicks.csv", header=True, schema=schema)

df = df.withColumn("ts", to_timestamp(col("Timestamp")))

df.printSchema()
df.show(5, truncate=False)
```
{% endraw %}
### Cálculo de Actividad por Usuario Mediante Ventanas Temporales de 1 Minuto en PySpark
{% raw %}
```python
from pyspark.sql.functions import window, sum as ssum

agg = (df.groupBy("User_ID", window(col("ts"), "1 minute"))
         .agg(ssum("Clicks").alias("clicks_min")))

agg.show(10, truncate=False)
```
{% endraw %}
### Cálculo del Top 10 de Usuarios con Mayor Actividad y Exportación a Pandas
{% raw %}
```python
from pyspark.sql.functions import desc, sum as ssum

top = (agg.groupBy("User_ID")
         .agg(ssum("clicks_min").alias("total_clicks"))
         .orderBy(desc("total_clicks"))
         .limit(10))

top.show()

# A Pandas para graficar
pdf = top.toPandas()
pdf
```
{% endraw %}
### Gráfico de Barras para Representar la Actividad de Usuarios en el Clickstream
{% raw %}
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
{% endraw %}
### Descarga Local del Gráfico Generado desde Google Colab

{% raw %}
```python
from google.colab import files
files.download("grafico_clicks.png")
```
{% endraw %}

<div style="text-align:center;">
  <img src="{{ site.baseurl }}/assets/grafico_clicks.png" alt="Gráfico de clics por usuario" width="80%">
</div>

<!-- ✅ Estilos y script para cuadros y botón copiar -->
<style>
pre {
  position: relative;
  padding: 14px 16px 16px 16px;
  border-radius: 10px;
  background: #0b1021;
  color: #e7e7e7;
  overflow: auto;
  box-shadow: 0 2px 8px rgba(0,0,0,.1);
}
pre.copy-wrap { padding-top: 42px; }
.copy-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  font-size: 12px;
  background: #1f6feb;
  color: #fff;
  border: none;
  padding: 6px 10px;
  border-radius: 6px;
  cursor: pointer;
}
.copy-btn:hover { background:#1158c7; }
.copy-btn.copied { background:#2ea043; }
.copy-btn.error { background:#d1242f; }
</style>
<script>
(function(){
function ready(f){/in/.test(document.readyState)?setTimeout(()=>ready(f),9):f();}
ready(function(){
document.querySelectorAll("pre").forEach(function(pre){
 if(pre.dataset.copyReady) return;
 pre.dataset.copyReady=1;
 pre.classList.add("copy-wrap");
 var btn=document.createElement("button");
 btn.className="copy-btn";
 btn.textContent="Copiar";
 btn.onclick=async function(){
   var code=pre.innerText;
   try{
     if(navigator.clipboard){await navigator.clipboard.writeText(code);} else {
       var t=document.createElement("textarea");t.value=code;document.body.appendChild(t);t.select();document.execCommand("copy");document.body.removeChild(t);
     }
     btn.textContent="¡Copiado!";btn.classList.add("copied");
     setTimeout(()=>{btn.textContent="Copiar";btn.classList.remove("copied");},1500);
   }catch(e){btn.textContent="Error";btn.classList.add("error");setTimeout(()=>{btn.textContent="Copiar";btn.classList.remove("error");},1500);}
 };
 pre.appendChild(btn);
});
});})();
</script>
