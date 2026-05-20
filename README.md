# Health-AirQuality_DataMining

Análisis de la relación entre **calidad del aire** y **salud respiratoria** en la
Región Metropolitana de Chile, 2020–2024. Integra datos de contaminantes
atmosféricos (PM10, PM2.5, NO₂, O₃, SO₂) con atenciones de urgencia, estadísticas
hospitalarias y defunciones, combinando EDA, estadística no paramétrica y modelos
de machine learning en torno a seis preguntas de investigación.

## Datos

| Dataset | Fuente |
|---|---|
| Calidad del aire | [MinCiencia / Datos-CambioClimatico](https://github.com/MinCiencia/Datos-CambioClimatico) |
| Urgencias, hospitalizaciones, defunciones | [DEIS / MINSAL](https://deis.minsal.cl/#datosabiertos) |

Todo se filtra a la Región Metropolitana (RM), 2020–2024. Los datos crudos no se
versionan (ver `data.7z`); las rutas y el filtrado están documentados en `CLAUDE.md`.

## Requisitos

Entorno Python (probado en Python 3.14) con:

```
pandas        # manejo de datos
numpy         # cálculo numérico
pyarrow       # lectura/escritura de los cachés Parquet (imprescindible)
scikit-learn  # MI, PCA, K-means, regresión logística, árbol de decisión
scipy         # Shapiro-Wilk, KS, Kruskal-Wallis, Spearman
statsmodels   # apoyo estadístico
matplotlib    # gráficos
seaborn       # gráficos
jupyterlab    # ejecución de los notebooks
```

Instalación rápida:

```bash
python -m venv env
source env/bin/activate
pip install pandas numpy pyarrow scikit-learn scipy statsmodels matplotlib seaborn jupyterlab
```

## Caché Parquet

Los CSV de urgencias pesan ~6 GB y tardan 10–20 min en leerse desde HDD. Tras la
primera carga, cada dataset se cachea a **Parquet** (de ahí la dependencia de
`pyarrow`); los notebooks detectan el caché y se saltan la lectura del CSV. La
invalidación es manual: borra el `.parquet` si cambian los datos o el filtrado.

## Notebooks

| Notebook | Contenido |
|---|---|
| `exploration_optimized.ipynb` | EDA y carga de cachés — ejecutar primero |
| `analysis_optimized.ipynb` | Modelado (`%run exploration_optimized.ipynb` al inicio) |
| `analisis_preguntas_investigacion.ipynb` | Flujo único estructurado por las 6 preguntas de investigación, con teoría de cada técnica |
| `EDA Respiratorio.ipynb` / `EDA_Respiratorio_update.ipynb` | EDA respiratorio hito 1 |

```bash
source env/bin/activate && jupyter lab   # o ./run_env.sh
```

## Hallazgos preliminares

Sobre 261 semanas alineadas (2020–2024):

- **Asociación contaminante–urgencias.** NO₂ (Spearman r = +0.27, p < 0.001) y SO₂
  (r = +0.16, p < 0.05) son los únicos contaminantes con asociación contemporánea
  significativa con las urgencias respiratorias. PM10, PM2.5 y O₃ no la alcanzan en
  lag 0.
- **Estacionalidad dominante.** Las urgencias varían fuertemente por estación
  (Kruskal-Wallis H = 110.6, p ≈ 10⁻²³), con picos invernales; la co-estacionalidad
  contaminación–salud es el principal confusor de la asociación bruta.
- **Efectos rezagados.** La correlación mejora con un desfase de **2–3 semanas** para
  NO₂ (r = +0.27 a 2 sem) y partículas (PM10 r = +0.15 a 3 sem), coherente con la
  cadena exposición → inflamación → consulta. O₃ no muestra asociación significativa
  con las urgencias en ningún lag.
- **Estructura de la contaminación.** Alta multicolinealidad PM10–PM2.5 (r = 0.81); el
  PCA separa dos sistemas: combustión/tráfico (PM + NO₂) e industrial/fotoquímico
  (SO₂ + O₃).
- **Predicción de semanas de alto riesgo.** Regresión logística y árbol de decisión
  superan el baseline en validación cruzada (F1 macro ≈ 0.71), pero el desempeño en
  test es más modesto (0.59–0.65) y captura en parte la estacionalidad.
- **Umbrales normativos.** Los días que superan la norma ICA «Malo»
  (PM10 > 100 µg/m³, PM2.5 > 37.5 µg/m³) presentan **~16–17 % más** urgencias
  respiratorias (p < 0.01).

> Limitaciones: el proxy de exposición (estación más cercana) introduce error de
> medición, no hay granularidad comunal, y los contrastes de umbral no controlan
> temperatura/estacionalidad. Las conclusiones causales requieren controles
> meteorológicos.
