# Pipeline de pronóstico de cultivos transitorios de gran escala en Colombia

Repositorio del pipeline completo de ciencia de datos desarrollado como parte del proyecto:

> **"Pronóstico de la producción de cultivos transitorios de gran escala en Colombia mediante series temporales y aprendizaje automático"**  
> Norbey Danilo Muñoz Cañón — Universidad Distrital Francisco José de Caldas, 2025-2026

---

## Descripción

Este repositorio implementa el pipeline de extremo a extremo para el análisis y pronóstico de la producción agrícola de cultivos transitorios en Colombia a nivel departamental (2007–2024), siguiendo la metodología CRISP-DM en tres fases:

**Fase 1 — Preparación de datos**  
Construcción del conjunto de datos analítico a partir de las Evaluaciones Agropecuarias Municipales (EVA) del Ministerio de Agricultura y Desarrollo Rural (MADR). Incluye limpieza, imputación, normalización, agregación departamental y aplicación del umbral de continuidad temporal. El resultado es un dataset de 490 pares departamento-cultivo con 15.077 observaciones semestrales (2007A–2024B).

**Fase 2 — Modelado y evaluación**  
Implementación completa del pipeline de modelado en dos componentes:

- *Modelado no supervisado*: segmentación de los 490 pares por escala productiva (K-Means, k=3, silueta=0,888) e identificación de los cinco pares de gran escala (Tolima-arroz, Casanare-arroz, Cundinamarca-papa, Boyacá-papa, Nariño-papa). Caracterización de patrones de dinámica temporal en cinco dimensiones (volatilidad, tendencia, estacionalidad, continuidad, concentración espacial).

- *Modelado supervisado*: pipeline progresivo de cuatro subfases (modelos baseline, modelos ML con configuración por defecto, ingeniería de características, optimización de hiperparámetros) con validación walk-forward expanding window de 11 folds. Familias de modelos: Seasonal Naive, SARIMA, ETS, Random Forest (RF), XGBoost (XGB), Prophet, Orbit DLT y LSTM.

**Fase 3 — Validación estadística y despliegue**  
Validación formal mediante prueba de Wilcoxon con coeficiente de rango biserial (r_rb), análisis de importancia de variables SHAP/permutación, y pronóstico hacia tres semestres futuros (2025A, 2025B, 2026A) bajo tres escenarios de área productiva.

### Productos del repositorio

| Producto | Descripción | Ubicación |
|---|---|---|
| Datasets procesados | Todos los datasets intermedios y finales del pipeline | `data/` |
| Resultados de validación | Métricas, predicciones y mejores hiperparámetros (JSON) | `models/` |
| Figuras EDA/clustering | PDF vectoriales generados por los notebooks de Fase 1 y 2 | `figures_eda_cleaning/` / `figures_clustering/` *(generadas localmente)* |
| Figuras supervisadas | PDF vectoriales generados por el notebook de Fase 2-3 | `figures_supervised/` *(generadas localmente)* |
| Anexo de tesis | Archivos anexos publicados | `outputs/` |

> **Nota sobre figuras:** las carpetas `figures_eda_cleaning/`, `figures_clustering/` y `figures_supervised/` no se incluyen en el repositorio por volumen (cientos de PDF vectoriales). Se generan automáticamente al ejecutar los notebooks.

---

## Estructura del repositorio

```
├── final_eda_cleaning.ipynb                          # Notebook Fase 1: EDA y limpieza
├── final_clustering.ipynb                            # Notebook Fase 2: clustering y segmentación
├── final_supervised.ipynb                            # Notebook Fase 2-3: pipeline supervisado y validación final
├── data/
│   ├── eva_2007_2018.csv                             # EVA fuente 2007–2018 (datos abiertos MADR)
│   ├── eva_2019_2024.csv                             # EVA fuente 2019–2024 (datos abiertos MADR)
│   ├── co.json                                       # GeoJSON Colombia para mapas (geopandas)
│   ├── transitorios.csv                              # Cultivos transitorios normalizados (municipal)
│   ├── transitorios_limpio_imp.csv                   # Con imputación de ceros
│   ├── transitorios_limpio_norm.csv                  # Con imputación y normalización de periodo
│   ├── transitorios_deptos.csv                       # Agregación departamental
│   ├── transitorios_deptos_fil_15.csv                # Filtro umbral ≥ 15 periodos
│   ├── transitorios_deptos_fil_18.csv                # Filtro umbral ≥ 18 periodos
│   ├── transitorios_deptos_fil_15_final.csv          # Umbral 15 + eliminación cultivos problemáticos
│   ├── transitorios_deptos_15_final_features.csv     # Con indicadores de clustering
│   ├── transitorios_deptos_15_final_features_cuartiles.csv    # Con asignación por cuartiles
│   ├── transitorios_deptos_15_final_features_segmentado.csv   # Con segmentación por escala
│   ├── transitorios_deptos_15_final_segmentacion_completa.csv # Con segmentación temporal completa
│   ├── series_gran_escala_imputadas.csv              # 5 pares gran escala con imputación 2018B
│   ├── registro_imputaciones.csv                     # Registro de imputaciones 2018B
│   └── series_gran_escala_maestro.csv                # Dataset final de series grandes (sin features)
├── models/                                           # Resultados JSON versionados (sin re-entrenamiento)
│   ├── baselines/                                    # Seasonal Naive, SARIMA, ETS
│   ├── ml_basico/                                    # RF, XGB, Prophet, Orbit DLT, LSTM (por defecto)
│   ├── ml_fe/                                        # RF, XGB, Prophet, Orbit DLT, LSTM (FE)
│   ├── RS_RF/                                        # Random Search — Random Forest
│   ├── GS_RF/                                        # Grid Search — Random Forest
│   ├── OPT_RF/                                       # Optuna TPE — Random Forest
│   ├── RS_XGB/                                       # Random Search — XGBoost
│   ├── GS_XGB/                                       # Grid Search — XGBoost
│   ├── OPT_XGB/                                      # Optuna TPE — XGBoost
│   ├── RS_LSTM/                                      # Random Search — LSTM
│   ├── GS_LSTM/                                      # Grid Search — LSTM
│   ├── OPT_LSTM/                                     # Optuna TPE — LSTM
│   └── estudios_optuna/                              # SQLite Optuna (excluido por .gitignore)
├── outputs/                                          # Anexo de la tesis
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Contenido de `models/` y comportamiento al clonar

### Qué se incluye en el repositorio

Todos los archivos `.json` de resultados están versionados. Cada archivo contiene las métricas de validación (MAPE, RMSE, MAE, R²), las predicciones fold a fold y, cuando aplica, los mejores hiperparámetros encontrados.

| Subcarpeta | Archivos incluidos | Contenido |
|---|---|---|
| `baselines/` | `resultados_snaive.json`, `resultados_sarima.json`, `resultados_ets.json` | Métricas y predicciones de los tres modelos baseline para los 5 pares |
| `ml_basico/` | `resultados_rf.json`, `resultados_xgb.json`, `resultados_prophet.json`, `resultados_dlt.json`, `resultados_lstm.json` | Métricas y predicciones de los modelos ML con configuración por defecto |
| `ml_fe/` | `resultados_rf_fe.json`, `resultados_xgb_fe.json`, `resultados_prophet_fe.json`, `resultados_dlt_fe.json`, `resultados_lstm_fe.json` | Métricas y predicciones con features enriquecidas |
| `RS_RF/`, `GS_RF/`, `OPT_RF/` | 5 archivos por par: `{depto}_{cultivo}_historial.json`, `{depto}_{cultivo}_metrics.json`, `{depto}_{cultivo}_params.json`, `{depto}_{cultivo}_predictions.json`, `{depto}_{cultivo}_semillas.json` | Historial de búsqueda, métricas, mejores hiperparámetros, predicciones fold a fold y métricas por semilla |
| `RS_XGB/`, `GS_XGB/`, `OPT_XGB/` | 5 archivos por par (mismo esquema) | Ídem para XGBoost |
| `RS_LSTM/`, `GS_LSTM/`, `OPT_LSTM/` | 5 archivos por par (mismo esquema) | Ídem para LSTM (incluye métricas por semilla de evaluación final) |
| `estudios_optuna/` | *(vacío — `.gitkeep`)* | La base de datos SQLite de Optuna se excluye por tamaño |

**No se incluyen** en el repositorio (excluidos por `.gitignore`):
- Pesos de modelos (`.joblib`, `.keras`, `.h5`) — se regeneran al re-entrenar
- Base de datos SQLite de estudios Optuna (`.db`) — se regenera con `load_if_exists=True`

### Qué ocurre al clonar y ejecutar

Al clonar el repositorio y ejecutar `final_supervised.ipynb`, **cada celda de entrenamiento verifica primero si existe el `.json` correspondiente**. Si existe, imprime `✓ Resultados cargados desde caché` y continúa sin entrenar. Si no existe, entrena y guarda.

El comportamiento esperado por subfase es el siguiente:

| Subfase | Al clonar (JSON presentes) | Desde cero (sin JSON) |
|---|---|---|
| Baselines (Seasonal Naive, SARIMA, ETS) | Carga instantánea | SARIMA y ETS tardan ~5–15 min por par |
| ML básico (RF, XGB, Prophet, DLT, LSTM) | Carga instantánea | RF y XGB: ~2 min; Prophet/DLT: ~5–10 min; LSTM: ~20–40 min por par |
| ML con features enriquecidas | Carga instantánea | Tiempos similares a ML básico |
| Random Search (RF, XGB) | Carga instantánea | ~15–30 min por modelo y par |
| Random Search (LSTM) | Carga instantánea | ~1–2 h por par |
| Grid Search (RF, XGB) | Carga instantánea | ~20–40 min por modelo y par |
| Grid Search (LSTM) | Carga instantánea | ~1–2 h por par |
| Optuna TPE (RF, XGB) | Carga instantánea | ~30–60 min por modelo y par |
| Optuna TPE (LSTM) | Carga instantánea | Hasta 30 min por par (timeout configurado) |

> Los tiempos estimados corresponden a CPU sin aceleración GPU. Con los JSON incluidos, la ejecución completa del notebook supervisado (sin re-entrenar) toma menos de 5 minutos.

---

## Datos fuente

Los datasets EVA provienen del portal de datos abiertos del Gobierno de Colombia:

- **EVA 2007–2018:** [Evaluaciones Agropecuarias Municipales](https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA/2pnw-mmge/about_data)
- **EVA 2019–2024:** [Evaluaciones Agropecuarias Municipales 2019–2024](https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA-2019-20/uejq-wxrr/about_data)

Los datasets derivados publicados están disponibles bajo licencia **CC BY 4.0**.

---

## Descripción de los datasets en `data/`

Los archivos se listan en el orden en que son generados por el pipeline:

- **`eva_2007_2018.csv`**: Dataset EVA de 2007 a 2018 de fuente de datos abiertos: https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA/2pnw-mmge/about_data
- **`eva_2019_2024.csv`**: Dataset EVA de 2019 a 2024 de fuente de datos abiertos: https://www.datos.gov.co/Agricultura-y-Desarrollo-Rural/Evaluaciones-Agropecuarias-Municipales-EVA-2019-20/uejq-wxrr/about_data
- **`transitorios.csv`**: Dataset inicial de solo cultivos transitorios (normalizado en nombres y a nivel municipal).
- **`transitorios_limpio_imp.csv`**: Dataset final de cultivos transitorios con imputación sobre los casos de ceros (análisis sistemático de valores 0 y su imputación).
- **`transitorios_limpio_norm.csv`**: Dataset final de cultivos transitorios con imputación de ceros y normalización sobre periodo.
- **`transitorios_deptos.csv`**: Dataset final de cultivos transitorios con imputación de ceros, normalización y agregación departamental.
- **`co.json`**: Fichero GeoJSON usado para construir el mapa de Colombia con geopandas.
- **`transitorios_deptos_fil_15.csv`** / **`transitorios_deptos_fil_18.csv`**: Dataset final de cultivos transitorios con imputación de ceros, normalización y agregación departamental, con filtros de umbral de periodos 15 y 18.
- **`transitorios_deptos_fil_15_final.csv`**: Dataset final de cultivos transitorios con imputación de ceros, normalización y agregación departamental, con filtro de umbral de periodos 15 y eliminación de "Cultivos problemáticos" (de denominación especial analizados).
- **`transitorios_deptos_15_final_features.csv`**: Dataset `transitorios_deptos_fil_15_final.csv` con las características construidas para alimentar el agrupamiento con indicadores de escala, volatilidad, tendencia, estacionalidad, continuidad y concentración.
- **`transitorios_deptos_15_final_features_cuartiles.csv`**: Dataset `transitorios_deptos_15_final_features.csv` con la asignación de pertenencia a uno de los cuartiles.
- **`transitorios_deptos_15_final_features_segmentado.csv`**: Dataset `transitorios_deptos_15_final_features_cuartiles.csv` con las nuevas columnas de segmentación: `cluster_escala`, `escala_productiva`.
- **`transitorios_deptos_15_final_segmentacion_completa.csv`**: Dataset `transitorios_deptos_15_final_features_segmentado.csv` con las nuevas columnas de segmentación temporal: `volatilidad`, `tendencia`, `estacionalidad`, `continuidad`, `concentracion`.
- **`series_gran_escala_imputadas.csv`**: Dataset de los pares de gran escala con imputación de periodo 2018B.
- **`registro_imputaciones.csv`**: Registro de las imputaciones 2018B realizadas.
- **`series_gran_escala_maestro.csv`**: Dataset de los pares de gran escala con imputación de periodo 2018B tanto para áreas como para producción. Dataset final de las series grandes sin features añadidas.

---

## Configuración del entorno

### Requisitos previos

- Python 3.10 o superior
- pip actualizado

### 1. Clonar el repositorio

```bash
git clone https://github.com/norbeydanilo/01-pipeline-pronostico-gran-escala.git
cd 01-pipeline-pronostico-gran-escala
```

### 2. Crear y activar el entorno virtual

**Windows:**
```bash
python -m venv .venv
.venv\Scripts\activate
```

**macOS / Linux:**
```bash
python -m venv .venv
source .venv/bin/activate
```

### 3. Instalar dependencias

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

> **Nota para GPU (opcional):** si se desea acelerar el entrenamiento de LSTM con GPU, reemplazar la línea de `tensorflow` en `requirements.txt` por `tensorflow[and-cuda]` y asegurarse de contar con los drivers CUDA correspondientes. El pipeline funciona íntegramente en CPU.

### 4. Registrar el entorno como kernel de Jupyter

```bash
pip install ipykernel
python -m ipykernel install --user --name=.venv --display-name "cultivos (venv)"
```

Abrir Jupyter y seleccionar el kernel **"cultivos (venv)"** antes de ejecutar cualquier notebook.

```bash
jupyter notebook
```

---

## Ejecución del pipeline

> Si los archivos `.json` de resultados ya existen en `models/` (incluidos en el repositorio), las celdas de entrenamiento los detectan automáticamente y **no re-entrenan**. Esto aplica a todos los modelos: baselines, ML por defecto, ML con features enriquecidas, Random Search, Grid Search y Optuna TPE.

### Orden de ejecución

| Notebook | Contenido | Genera |
|---|---|---|
| `final_eda_cleaning.ipynb` | EDA, limpieza, imputación, agregación departamental | Datasets en `data/`, figuras en `figures_eda_cleaning/` |
| `final_clustering.ipynb` | Segmentación por escala, caracterización temporal | Datasets en `data/`, figuras en `figures_clustering/` |
| `final_supervised.ipynb` | Modelos baseline, ML, FE, optimización, Wilcoxon, SHAP, pronóstico | Resultados en `models/`, figuras en `figures_supervised/` |

Cada notebook es independiente si los datasets de entrada ya están en `data/`. Para una ejecución completa desde cero, ejecutar en el orden indicado.

### Regenerar solo figuras (sin re-entrenar)

Al clonar el repositorio con los `.json` incluidos, para generar únicamente las figuras del notebook supervisado:

1. Ejecutar **Celda 0** — define `save_sup()` y crea carpetas *(siempre primero)*
2. Ejecutar **celda de configuración** — imports, constantes, funciones auxiliares
3. Ejecutar **celdas de carga de datos** — series imputadas, datasets FE
4. Ejecutar **celdas de figuras** directamente — los modelos se cargan desde JSON automáticamente

---

## Licencia

Este repositorio se publica bajo licencia **MIT**.

Eres libre de usar, copiar, modificar, fusionar, publicar, distribuir, sublicenciar y/o vender copias del software y sus materiales asociados, con o sin modificación, para cualquier propósito incluyendo uso académico, investigativo y comercial, siempre que se mantenga el aviso de autoría original.

Los datasets derivados del pipeline están disponibles bajo licencia **CC BY 4.0** (Creative Commons Atribución 4.0 Internacional). Puedes compartirlos y adaptarlos para cualquier propósito, incluso comercial, siempre que se cite la fuente.

### Uso y replicación

Este repositorio está diseñado para ser completamente replicable y extensible:

- **Replicación total**: clonar el repositorio, instalar dependencias y ejecutar los notebooks en el orden indicado reproduce todos los resultados del estudio.
- **Extensión metodológica**: el pipeline está estructurado por fases independientes. Es posible incorporar nuevos modelos, nuevos cultivos, otras regiones geográficas o periodos temporales distintos modificando las celdas de configuración de cada notebook.
- **Reutilización parcial**: cada subfase (EDA, clustering, baseline, optimización, validación estadística) puede ejecutarse de forma aislada si los datos de entrada están disponibles.
- **Investigación derivada**: se invita a investigadores, estudiantes y profesionales del sector agropecuario a extender, criticar y mejorar este trabajo. Los resultados, código y datos son de libre acceso.

---

## Autoría

**Norbey Danilo Muñoz Cañón**  
Facultad de Ingeniería.  
Maestría en Ciencias de la Información y las Comunicaciones.  
Grupo de Investigación INTECSE - Interoperabilidad Tecnológica y Semántica.  
Universidad Distrital Francisco José de Caldas.  
Bogotá, Colombia — 2025/2026.  

---

> *Created by Norbey Danilo Muñoz Cañón, 2025/26.*  
> *The idea of intellectual property is fundamentally wrong. Knowledge belongs to all people!*
