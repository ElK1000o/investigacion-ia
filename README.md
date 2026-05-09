# Inserción Laboral de Migrantes en Chile

Trabajo 1 — ICCD1021 Taller de Ciencia de Datos V: Inteligencia Artificial  
Universidad Mayor · Semestre 2026-01

---

## Descripción

Análisis de las brechas de inserción laboral entre población migrante y chilena utilizando datos de la **Encuesta Nacional de Empleo (ENE) 2024** y la **Encuesta de Caracterización Socioeconómica Nacional (CASEN) 2024**.

Se modelan tres hipótesis centrales:
- Los migrantes presentan menores tasas de formalidad laboral controlando por características observables.
- Existe una brecha salarial significativa (ingreso/hora) entre migrantes y chilenos.
- Los migrantes se concentran en perfiles laborales de menor calidad de inserción (baja puntuación IILM).

---

## Estructura del proyecto

```
Trabajo 1/
├── input/
│   └── data/
│       ├── ene_2024.csv          # ENE 2024 — INE Chile (390K filas, 185 vars)
│       └── casen_2024.dta        # CASEN 2024 — MDS Chile (218K filas, 877 vars)
├── output/
│   ├── data/
│   │   ├── ene_procesada.parquet
│   │   └── casen_procesada.parquet
│   └── fig/
│       └── fig01–fig21 *.png
├── Python/
│   ├── generar_notebooks.py      # Genera los 5 notebooks a partir de este script
│   ├── Procesamiento/
│   │   ├── 01_ene_preprocesamiento.ipynb
│   │   └── 02_casen_preprocesamiento.ipynb
│   └── Analisis/
│       ├── 03_eda_descriptivo.ipynb
│       ├── 04_modelos_supervisados.ipynb
│       └── 05_clustering_pca.ipynb
├── Informe/                      # Informe en LaTeX (pendiente)
├── Instrucciones/
├── Ejemplos/
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
cd Python
python generar_notebooks.py

# 2. Ejecutar en orden
jupyter nbconvert --to notebook --execute --inplace Procesamiento/01_ene_preprocesamiento.ipynb
jupyter nbconvert --to notebook --execute --inplace Procesamiento/02_casen_preprocesamiento.ipynb
jupyter nbconvert --to notebook --execute --inplace Analisis/03_eda_descriptivo.ipynb
jupyter nbconvert --to notebook --execute --inplace Analisis/04_modelos_supervisados.ipynb
jupyter nbconvert --to notebook --execute --inplace Analisis/05_clustering_pca.ipynb
```

O abrir directamente en Jupyter y ejecutar celda a celda.

---

## Notebooks

| # | Notebook | Contenido | Salidas |
|---|----------|-----------|---------|
| 01 | `01_ene_preprocesamiento.ipynb` | Limpieza ENE, variable migrante, formalidad, sobrecalificación | `ene_procesada.parquet` |
| 02 | `02_casen_preprocesamiento.ipynb` | Limpieza CASEN, ingreso/hora, `educa` reconstruida, IILM | `casen_procesada.parquet` |
| 03 | `03_eda_descriptivo.ipynb` | EDA comparativo, Pareto de ingresos, Spearman, T-test | fig01–fig12 |
| 04 | `04_modelos_supervisados.ipynb` | OLS, Logística, Árbol, Random Forest, ROC, odds ratio | fig13–fig16 |
| 05 | `05_clustering_pca.ipynb` | PCA, K-Means, DBSCAN (comparativo), perfiles por cluster | fig17–fig21 |

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

## Nota metodológica

El **IILM** es un índice de elaboración propia para este trabajo, no un índice oficial. Se inspira metodológicamente en el Índice de Pobreza Multidimensional (IPM) que utiliza Chile en la CASEN, adaptado para medir calidad de inserción laboral.
