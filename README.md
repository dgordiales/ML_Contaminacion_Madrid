# ML_Contaminacion_Madrid
Repositorio del proyecto de Machine Learning realizado por alumnos del Bootcamp online Data Science &amp; IA de The Bridge


### ⚠️ Nota importante

# Los CSV originales de datos abiertos se excluyen por tamaño (superan el limite de GitHub).
# Existen localmente en la rama feature/eda dentro de src/data_sample/, pero no se suben al repo.
src/data_sample/calidad_del_aire.csv
src/data_sample/datos_meteorologicos.csv
src/data_sample/estaciones_control_aire.csv
src/data_sample/estaciones_medicion_trafico.csv
src/data_sample/trafico_diario.csv

El **`dataset_final.csv`** (ya procesado y unido) sí está incluido en el repositorio, y es el punto de partida para el resto del pipeline (preprocesado, modelado, evaluación).

### Dataset final

- **Filas**: 51.001 (una fila = estación de medición × día)
- **Periodo**: 2019-2024
- **Estaciones**: 24 estaciones de calidad del aire de Madrid
- **Target**: `no2` (nivel de NO2, µg/m³)
- **Features**: `temperatura`, `humedad`, `viento_vel`, `precipitacion`, `intensidad_mean`, `ocupacion_mean`, `carga_mean`, `vmed_mean`