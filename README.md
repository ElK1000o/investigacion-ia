# Inserción Laboral de Migrantes en Chile

Trabajo 1 — ICCD1021 Taller de Ciencia de Datos V: Inteligencia Artificial  
Universidad Mayor · Semestre 2026-01  
Autores: Benjamín Infante · Camilo Riquelme

---

## Descripción

Análisis de las brechas de inserción laboral entre población migrante y chilena utilizando datos de la **Encuesta Nacional de Empleo (ENE) 2024** y la **Encuesta de Caracterización Socioeconómica Nacional (CASEN) 2024**.

Se modelan seis hipótesis centrales:

- **H1. Ocupación:** Los migrantes presentan mayor participación y ocupación laboral que la población nacional.
- **H2. Brecha salarial:** Los migrantes tienen menores ingresos por hora, incluso controlando por educación, edad, sexo y macrozona.
- **H3. Informalidad:** Los migrantes presentan mayor informalidad y menor acceso a protección social.
- **H4. Segmentación ocupacional:** Los migrantes se concentran en ocupaciones de menor valorización.
- **H5. Heterogeneidad migrante:** La inserción migrante no es homogénea; existen perfiles diferenciados identificables por clustering.
- **H6. Subempleo calificado:** Existe desajuste entre nivel educativo y ocupación desempeñada.

---

## Estructura del proyecto

```
investigacion-ia/
├── input/
│   └── data/
│       ├── ene_2024.csv          # ENE 2024 — INE Chile (390K filas, 185 vars)
│       └── casen_2024.dta        # CASEN 2024 — MDS Chile (218K filas, 877 vars)
├── output/
│   └── data/
│       ├── ene_procesada.parquet
│       ├── ene_procesada.csv
│       ├── casen_procesada.parquet
│       └── casen_procesada.csv
├── Python/
│   ├── Procesamiento/
│   │   ├── 01_ene_preprocesamiento.ipynb
│   │   └── 02_casen_preprocesamiento.ipynb
│   └── Analisis/
│       ├── 03_eda_descriptivo.ipynb
│       ├── 04_modelos_supervisados.ipynb
│       └── 05_clustering_pca.ipynb
├── Informe/
│   ├── Main.tex                  # Documento maestro LaTeX
│   ├── Main.pdf                  # Informe compilado
│   ├── Referencias.bib           # Bibliografía BibTeX
│   ├── Figuras/
│   │   └── visualizaciones/      # fig01–fig24 generados por los notebooks
│   ├── Portada/
│   │   └── portada.tex
│   └── Proyecto/
│       └── proyecto.tex          # Cuerpo del informe
├── Instrucciones/
├── Ejemplos/
├── generar_notebooks.py          # Regenera los 5 notebooks desde este script
├── interpretaciones_graficos.md  # Análisis figura a figura
└── README.md
```

---

## Datos

Los archivos de datos crudos **no están incluidos en el repositorio** por restricciones de tamaño y licencia.

| Fuente | Archivo | Acceso |
|--------|---------|--------|
| ENE 2024 (INE Chile) | `ene_2024.csv` | [ine.cl](https://www.ine.gob.cl/estadisticas/sociales/mercado-laboral/ocupacion-y-desocupacion) |
| CASEN 2024 (MDS) | `casen_2024.dta` | [observatorio.ministeriodesarrollosocial.gob.cl](http://observatorio.ministeriodesarrollosocial.gob.cl/encuesta-casen) |

Una vez descargados, colocarlos en `input/data/` con los nombres indicados.

---

## Cómo ejecutar

### Requisitos

```
python >= 3.10
pandas
numpy
matplotlib
seaborn
scipy
scikit-learn
statsmodels
pyarrow        # para leer/escribir parquet
pyreadstat     # para leer .dta (CASEN)
jupyter
```

Instalar dependencias:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-learn statsmodels pyarrow pyreadstat jupyter
```

### Orden de ejecución

Los notebooks **deben ejecutarse en orden**: el procesamiento genera los `.parquet` que consumen los análisis.

```bash
# 1. (Opcional) Regenerar los notebooks desde el script fuente
python generar_notebooks.py

# 2. Ejecutar en orden
jupyter nbconvert --to notebook --execute --inplace Python/Procesamiento/01_ene_preprocesamiento.ipynb
jupyter nbconvert --to notebook --execute --inplace Python/Procesamiento/02_casen_preprocesamiento.ipynb
jupyter nbconvert --to notebook --execute --inplace Python/Analisis/03_eda_descriptivo.ipynb
jupyter nbconvert --to notebook --execute --inplace Python/Analisis/04_modelos_supervisados.ipynb
jupyter nbconvert --to notebook --execute --inplace Python/Analisis/05_clustering_pca.ipynb
```

---

## Notebooks

| # | Notebook | Contenido | Salidas |
|---|----------|-----------|---------|
| 01 | `01_ene_preprocesamiento.ipynb` | Limpieza ENE, variable migrante, formalidad, sobrecalificación | `ene_procesada.parquet` |
| 02 | `02_casen_preprocesamiento.ipynb` | Limpieza CASEN, ingreso/hora, `educa` reconstruida, IILM | `casen_procesada.parquet` |
| 03 | `03_eda_descriptivo.ipynb` | EDA comparativo, Pareto de ingresos, Spearman, T-test | fig01–fig12 |
| 04 | `04_modelos_supervisados.ipynb` | OLS, Logística, Árbol, Random Forest, ROC, odds ratio | fig13–fig16 |
| 05 | `05_clustering_pca.ipynb` | PCA, K-Means, DBSCAN (comparativo), perfiles por cluster | fig17–fig24 |

---

## Variables clave construidas

**ENE:**
- `migrante` — `(nacionalidad != 152)`: 1 si la persona no es de nacionalidad chilena
- `formal` — `ocup_form == 1`: empleo formal según ENE
- `ponderador` — `fact_anual` convertido a float (separador decimal era coma)

**CASEN:**
- `migrante` — `(lugar_nac == 1)`: 1 si nació fuera de Chile
- `ingreso_hora` — `yoprcor / (o10 × 4.33)`: ingreso laboral por hora trabajada, winsorizado al p99
- `educa` — reconstruida desde `e6a_no_asiste + e6c_completo`: escala 1–7 con distinción técnica/universitaria completa e incompleta
- `formal` — `contrato OR cotiza`: trabajador con contrato escrito o que cotiza en previsión
- `iilm` — Índice de Inserción Laboral Multidimensional (construcción propia, inspirada en enfoque Alkire-Foster): promedio de 5 dimensiones binarias (empleo, formalidad, ingreso, protección, adecuación)

---

## Hallazgos principales

- Los migrantes presentan mayor tasa de ocupación (+20,7 pp) pero menor formalidad (−6,1 pp) y menor ingreso/hora (−8,3% en mediana; diferencia bruta de medias bootstrap: 579 CLP/h, IC 95%: [389, 772]; brecha ajustada OLS: ~9,4%, coef. −0,099, p<0,001).
- El modelo OLS confirma que la brecha salarial persiste al controlar por educación, edad, sexo y macrozona.
- La regresión logística y el Random Forest (AUC ~0,89) confirman menor probabilidad de formalidad para migrantes; el sector económico es el predictor más relevante.
- K-Means (K=3) identifica tres perfiles laborales: el Cluster 0 concentra 38,2% de migrantes (vs 9,1% global) y combina la educación más alta del grupo (5,6/7) con el ingreso más bajo (3.017 CLP/h), evidencia directa de subempleo calificado.

---

## Nota metodológica

El **IILM** es un índice de elaboración propia para este trabajo, no un índice oficial. Se inspira metodológicamente en el Índice de Pobreza Multidimensional (IPM) que utiliza Chile en la CASEN, adaptado para medir calidad de inserción laboral.

La selección del año 2024 responde a que la CASEN 2024 es la edición más reciente disponible. Dado que el análisis de ingresos requiere información de esa encuesta, se utilizó el mismo año para la ENE con el fin de mantener concordancia temporal entre ambas fuentes.

---

## Informe

El informe completo está disponible en `Informe/Main.pdf`. Compilado en LaTeX con bibliografía en formato APA (`apacite`).