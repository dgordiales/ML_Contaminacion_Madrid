# ML_Contaminacion_Madrid

Proyecto de Machine Learning realizado por alumnos del Bootcamp online **Data Science & IA** de **The Bridge**.

El objetivo es **predecir la concentración diaria de NO₂ (µg/m³)** en cada estación de calidad del aire de Madrid, a partir de variables **meteorológicas, de tráfico y temporales**, para anticipar jornadas con niveles elevados de contaminación.

---

## Tabla de contenidos

- [Descripción del problema](#descripción-del-problema)
- [Datos](#datos)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Pipeline del proyecto](#pipeline-del-proyecto)
- [Metodología y prevención de data leakage](#metodología-y-prevención-de-data-leakage)
- [Resultados](#resultados)
- [Limitaciones](#limitaciones)
- [Mejoras futuras](#mejoras-futuras)
- [Cómo reproducir](#cómo-reproducir)
- [Stack técnico](#stack-técnico)

---

## Descripción del problema

El NO₂ es uno de los contaminantes atmosféricos más vigilados en entornos urbanos por su impacto en la salud. Se plantea un problema de **regresión**: estimar el nivel diario de NO₂ de cada estación combinando fuentes de datos abiertos del Ayuntamiento de Madrid (calidad del aire, tráfico y meteorología).

El escenario de predicción es de **seguimiento diario operativo**: predecir el NO₂ de una estación en un día usando información disponible hasta ese momento (incluida la última observación previa de NO₂).

---

## Datos

### Dataset final

- **Filas**: 51.001 (una fila = estación de medición × día)
- **Periodo**: 2019–2024
- **Estaciones**: 24 estaciones de calidad del aire de Madrid
- **Target**: `no2` (nivel de NO₂, µg/m³)
- **Features base**: `temperatura`, `humedad`, `viento_vel`, `precipitacion`, `intensidad_mean`, `ocupacion_mean`, `carga_mean`, `vmed_mean`

> ⚠️ **Nota sobre los CSV originales**
> Los CSV originales de datos abiertos se excluyen por tamaño (superan el límite de GitHub). Existen localmente en la rama `feature/eda` dentro de `src/data_sample/`, pero no se suben al repo:
> ```
> src/data_sample/calidad_del_aire.csv
> src/data_sample/datos_meteorologicos.csv
> src/data_sample/estaciones_control_aire.csv
> src/data_sample/estaciones_medicion_trafico.csv
> src/data_sample/trafico_diario.csv
> ```
> El **`dataset_final.csv`** (ya procesado y unido) sí está incluido en el repositorio, y es el punto de partida para el resto del pipeline (preprocesado, modelado, evaluación).

---

## Estructura del repositorio

```
ML_Contaminacion_Madrid/
├── README.md
├── main.ipynb                             # Notebook completo (6 partes, de datos a evaluación)
├── src/
│   ├── data_sample/                       # Datasets (CSV originales solo en local)
│   │   └── dataset_final.parquet
│   │   └── test_fe.parquet
│   │   └── test.parquet
│   │   └── train_fe.parquet
│   │   └── train.parquet
│   └── models/                            # Pipeline final serializado (.pkl)
│   │   └── modelo_ridge_no2_metadata.json
│   │   └── modelo_ridge_no2_metadata.joblib                         
└── notebooks/                             # Notebooks originales por fase (opcional)
    ├── 01_Creaccion_Dataset_final.ipynb
    ├── 02_Mini_EDA_Inicial.ipynb
    ├── 03_Analisis_Target_NO2.ipynb
    ├── 04_Correlaciones_Outliers_Features.ipynb
    ├── preprocessing.ipynb
    └── 05_Modelado_Optimizacion_Evaluacion.ipynb
```

El **notebook unificado** reproduce todo el proceso de principio a fin. Los notebooks originales por fase se conservan como referencia.

---

## Pipeline del proyecto

El notebook unificado está organizado en 6 partes que siguen el orden lógico del proyecto:

| Parte | Contenido |
|-------|-----------|
| **1. Creación del dataset final** | Unión de los 5 ficheros originales: paso de calidad del aire y meteorología de formato ancho a largo, corrección de escalas, emparejamiento geográfico (estación de aire ↔ punto de tráfico más cercano por distancia haversine) y unión final. |
| **2. Mini-EDA inicial** | Tipos, nulos, estadísticos descriptivos, tabla de variables y **división train/test**. |
| **3. Análisis del target (NO₂)** | Distribución (simetría/asimetría), outliers (IQR y Z-score), variación temporal (mes, día de la semana, findes vs. laborables), variación por estación y relación con tráfico y meteorología. |
| **4. Correlaciones entre features y outliers** | Heatmap de correlación entre features, detección automática de multicolinealidad, pairplots y análisis de outliers en tráfico y meteorología. |
| **5. Feature engineering (preprocesado)** | Split cronológico, **features temporales**, **lags y medias móviles de NO₂**, **imputación de meteo con flag de missing** y **target encoding de estación**, encadenados en un `Pipeline` de transformers propios. |
| **6. Modelado, optimización y evaluación** | Métrica principal (MAE), validación cruzada temporal, baseline y comparación de modelos, optimización de Ridge e HistGradientBoosting, evaluación final sobre test, análisis de errores, interpretabilidad y serialización del pipeline. |

---

## Metodología y prevención de data leakage

- **Split cronológico** (no aleatorio): el corte train/test se hace por fecha para respetar la naturaleza temporal del problema y evitar que el modelo "vea el futuro".
- **Validación cruzada temporal**: los folds respetan el orden temporal.
- Todo el **feature engineering que depende del target** (target encoding, lags/medias móviles) se ajusta **solo con train** y se aplica a test.
- El modelo, las variables y los hiperparámetros se fijan **antes** de tocar el conjunto de test; el test se usa una única vez para la evaluación final.

---

## Resultados

- **Métrica principal**: **MAE** (error medio en µg/m³ de NO₂), acompañada de **RMSE** y **R²** como secundarias.
- **Modelo seleccionado**: **Ridge (`alpha = 1000`)**, elegido por:
  - menor MAE medio,
  - menor variabilidad entre folds temporales,
  - mayor rapidez y facilidad de interpretación,
  - ausencia del sobreajuste train/validación observado en el boosting.
- El pipeline final se serializa con `joblib` para su reutilización.

**Conclusiones de negocio**: el modelo permite anticipar jornadas con concentraciones elevadas de NO₂, apoyar la planificación de medidas preventivas, identificar patrones meteorológicos/temporales/espaciales y complementar la monitorización cuando existan retrasos en los datos.

---

## Limitaciones

1. El modelo necesita una observación previa de NO₂, por lo que no predice el primer registro de cada estación en un bloque independiente.
2. `no2_lag1` es la observación anterior *disponible*, que no siempre corresponde al día anterior exacto cuando hay huecos en las fechas.
3. El modelo captura **asociaciones estadísticas**, no relaciones causales.
4. Algunas variables de tráfico presentan calidad/cobertura irregular.

---

## Mejoras futuras

- Incorporar más fuentes (calendario de festivos, eventos, más variables meteorológicas).
- Probar modelos específicos de series temporales.
- Ampliar el histórico y mejorar la cobertura de tráfico.
- Desplegar el pipeline en un flujo de predicción diaria.

---

## Cómo reproducir

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/dgordiales/ML_Contaminacion_Madrid
   cd ML_Contaminacion_Madrid
   ```
2. Crear el entorno e instalar dependencias:
   ```bash
   pip install pandas numpy scikit-learn matplotlib seaborn joblib pyarrow jupyter
   ```
3. Colocar `dataset_final.csv` (incluido) en `src/data_sample/`. Los CSV originales solo son necesarios para reejecutar la **Parte 1**.
4. Abrir y ejecutar el notebook:
   ```bash
   main.ipynb
   ```
   Ejecutar las celdas en orden (Parte 1 → 6).

---

## Stack técnico

`Python` · `pandas` · `numpy` · `scikit-learn` · `matplotlib` · `seaborn` · `joblib` · `pyarrow`

---

*Proyecto formativo — Bootcamp Data Science & IA, The Bridge.*
