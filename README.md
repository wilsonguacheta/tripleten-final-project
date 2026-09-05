# Proyecto Final de Analítica de Datos

Tres casos de uso independientes, cada uno con su notebook y su presentación.

| Caso | Notebook | Presentación |
|---|---|---|
| **Principal** — CallMeMaybe: operadores ineficaces | [`main_project/01_decomposicion.ipynb`](main_project/01_decomposicion.ipynb) (plan)<br>[`main_project/02_analisis_operadores.ipynb`](main_project/02_analisis_operadores.ipynb) (análisis) | `main_project/presentacion/` |
| **Test A/B** — sistema de recomendaciones | [`ab_project/ab_test_analysis.ipynb`](ab_project/ab_test_analysis.ipynb) | `ab_project/presentacion/` |
| **SQL** — servicio de libros | [`sql_project/sql_analysis.ipynb`](sql_project/sql_analysis.ipynb) | `sql_project/presentacion/` |

El **dashboard** del caso principal se construye en Tableau Public siguiendo [`main_project/dashboard/GUIA_TABLEAU.md`](main_project/dashboard/GUIA_TABLEAU.md), con los datos ya preparados en `main_project/dashboard/tableau_extract.csv`.

---

## Recrear el entorno

Todo el proyecto se ejecuta en un entorno conda declarado en `environment.yml`. Desde esta carpeta:

```bash
conda env create -f environment.yml
```

```bash
conda run -n dataanalyst-final python -m ipykernel install --user --name dataanalyst-final --display-name "Python (dataanalyst-final)"
```

Los cuatro notebooks están guardados con el kernel `dataanalyst-final`. Al abrirlos en Jupyter, selecciona ese kernel si no aparece ya seleccionado.

```bash
conda activate dataanalyst-final
```

```bash
jupyter lab
```

## Volver a ejecutar todo

```bash
conda run -n dataanalyst-final jupyter nbconvert --to notebook --execute --inplace main_project/01_decomposicion.ipynb main_project/02_analisis_operadores.ipynb ab_project/ab_test_analysis.ipynb sql_project/sql_analysis.ipynb
```

El notebook de SQL requiere conexión a internet: consulta una base PostgreSQL remota.

## Regenerar las presentaciones

Las presentaciones se construyen a partir de los gráficos de los notebooks ya ejecutados, de modo que nunca se desincronizan del análisis:

```bash
conda run -n dataanalyst-final python build_presentaciones.py
```

Genera un `.pptx` editable y su `.pdf` en la carpeta `presentacion/` de cada caso. La conversión a PDF usa PowerPoint y solo funciona en Windows; sin él, exporta los `.pptx` a mano.

---

## Organización de los datos

```
data/
├── raw/        # datos originales — no se modifican nunca
├── interim/    # datos limpios intermedios
└── processed/  # tablas agregadas y resultados
```

Los notebooks solo leen de `raw/` y escriben en `interim/` y `processed/`. Los archivos generados pueden borrarse sin pérdida: se reconstruyen al ejecutar los notebooks.

## Nota sobre las credenciales

El notebook de SQL lee la conexión a la base de datos del curso desde variables de entorno (`sql_project/.env`, no versionado) vía `python-dotenv`. Copia `sql_project/.env.example` a `sql_project/.env` y complétalo con las credenciales que TripleTen proporciona en el enunciado para poder ejecutar el notebook.
