---
layout: default
title: "Análisis de Flujo de Datos Simulado con Spark"
---

{% include nav.html %}
<link rel="stylesheet" href="{{ site.baseurl }}/assets/style.css">

# 📊 Análisis de Flujo de Clickstream con PySpark

Este proyecto tiene como objetivo procesar un flujo simulado de clics para identificar patrones de navegación de usuarios utilizando **PySpark** y técnicas de **ventanas temporales**.

Es parte del ejercicio académico de *Big Data & Analytics*, donde se busca comprender cómo Spark procesa series de tiempo y aplica agregaciones por intervalos.

---

## 🎯 Objetivo del análisis

- Simular un dataset de **clickstream**
- Procesarlo con **PySpark**
- Aplicar **ventanas de 1 minuto**
- Obtener el **Top 10 usuarios con más clics**
- Visualizar el resultado y **analizar patrones**

---

## 📂 Datos utilizados

El dataset contiene 2000 eventos de clics con las columnas:

| Campo | Descripción |
|-------|------------|
| `Timestamp` | Fecha y hora del clic |
| `User_ID`   | Identificador del usuario (U001...U030) |
| `Clicks`    | Cantidad de clics registrados |

Si el sistema no proporciona datos, estos se pueden simular (lo hicimos en este caso).

---

## ⚙️ Entorno de trabajo

- **Google Colab**
- **PySpark 3.5.1**
- **Python 3.x**
- **Matplotlib** para graficar

Instalación:

```python
!pip install pyspark==3.5.1

from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Clickstream").getOrCreate()
