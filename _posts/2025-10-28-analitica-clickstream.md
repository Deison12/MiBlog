---
layout: default
title: "Análisis de Flujo de Datos Simulado con Spark"
---

## Objetivo
Procesar un flujo de clics para detectar patrones de navegación y su aporte al negocio.

## Datos
CSV con columnas `Timestamp, User_ID, Clicks`. (Si el aula no provee un CSV, generaremos uno simulado).

## Configuración
**Entorno sugerido:** Google Colab + PySpark 3.5.1.  
Ejecutar:
```python
!pip install pyspark==3.5.1
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Clickstream").getOrCreate()
```
> Nota: Ajusta versión si tu entorno ya tiene PySpark.

## Proceso (ventanas de 1 minuto)
Transformación clave con ventana temporal de 1 minuto usando `window(ts, "1 minute")`.  
Fragmento (se completará en el Paso 3):
```python
from pyspark.sql.functions import to_timestamp, col, window, sum as ssum
df = spark.read.csv("/content/clicks.csv", header=True, schema="Timestamp STRING, User_ID STRING, Clicks INT")
df = df.withColumn("ts", to_timestamp(col("Timestamp")))
agg = (df.groupBy("User_ID", window(col("ts"), "1 minute"))
         .agg(ssum("Clicks").alias("clicks_min")))
```

## Visualización
Se exportará el Top 10 de usuarios a Pandas y se graficará en barras con Matplotlib.  
Coloca aquí la imagen exportada:  
`![Gráfico de clics](../assets/grafico_clicks.png)`

## Hallazgos (borrador)
- Picos de clics entre HH:MM–HH:MM.  
- ~20% de usuarios concentran ~80% de los clics (regla 80/20).  
- Usuarios U00X/U00Y muestran sesiones intensas → oportunidades de personalización.  

## Despliegue del Blog y Arquitectura
- **Arquitectura:** Jekyll (estático) + tema Cayman (remote_theme) + GitHub Pages.
- **Pipeline de contenido:** Markdown → build → publicación automática.
- **Funciones:** navegación, post con código, imagen del gráfico, enlaces al dataset.

## Streaming vs. Lotes (breve)
- **Batch:** datos en reposo; ideal para informes y cierres diarios.  
- **Streaming:** datos en movimiento; ventanas temporales, baja latencia, decisiones casi en tiempo real.  
- **Conclusión:** el streaming es clave cuando necesitamos reaccionar al comportamiento del usuario al vuelo.

## Cierre
Este post se actualizará con los resultados y la gráfica del análisis (Pasos 2–4).
<div style="text-align:center;">
  <img src="{{ site.baseurl }}/assets/grafico_clicks.png" alt="Gráfico de clics por usuario" width="80%">
</div>

