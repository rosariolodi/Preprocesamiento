# NovaMarket – Auditoría de Calidad de Datos

Proyecto del curso de Preprocesamiento de Datos (5to semestre). El objetivo es realizar una auditoría integral de calidad de datos sobre el dataset **NovaMarket** (~640,000 filas, 20 columnas), evaluando las seis dimensiones DAMA: **Completitud, Exactitud, Consistencia, Validez, Unicidad y Oportunidad**.

## Integrantes

- [Rosario lozano]
- [Laura Torres]
- [Diego Diaz]

## Estructura del repositorio

## Dataset

El archivo `NovaMarket_datos_crudos.csv` **no está incluido en el repositorio** por su tamaño. Descárgalo desde:

[LINK AQUÍ]

Colócalo en la carpeta `data/` antes de ejecutar el notebook.

## Entorno de trabajo

- Python 3.10
- Entorno Conda: `Analitica_sweaty`
- Librerías principales: `pandas`, `matplotlib`, `seaborn`, `openpyxl`

Para crear el entorno:

```bash
conda create -n Analitica_sweaty python=3.10
conda activate Analitica_sweaty
pip install pandas matplotlib seaborn openpyxl jupyter
```

## Flujo de trabajo en equipo

1. Antes de empezar: `git pull origin main`
2. Trabajar en ramas separadas por dimensión o tarea:
```bash
   git checkout -b nombre-tu-rama
```
3. Commits descriptivos por avance:
```bash
   git add .
   git commit -m "Completitud: análisis MCAR/MAR nivel_satisfaccion"
   git push origin nombre-tu-rama
```
4. Crear Pull Request hacia `main` y revisar cambios en conjunto antes de fusionar.

## Dimensiones DAMA cubiertas

- [x] Completitud
- [x] Exactitud
- [x] Consistencia
- [x] Validez
- [x] Unicidad
- [x] Oportunidad

## Entregables

- Notebook de auditoría: `S03_PD_Apellido_AuditoriaCalidad.ipynb`
- Catálogo de hallazgos (Excel)
- Plan de saneamiento
- Video de presentación (~4 min)