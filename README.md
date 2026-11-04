# Distribución de la propiedad horizontal en las comunas de Medellín

Proyecto individual de ciencia de datos del programa Tecnólogo en Análisis y Desarrollo de Software (SENA), ficha 3223873.
Esta versión contiene el **diagnóstico inicial de calidad de datos** (evidencia GA1-220501094-01-AA1-EV06).

**Autor:** Maicol Esneider Posada Callejas · **Instructor:** Carlos Navia

## Descripción

El proyecto analiza la información catastral abierta de Medellín para saber dónde se concentran los predios en propiedad horizontal (conjuntos residenciales y edificios de apartamentos). En esta etapa se cargan, inspeccionan y diagnostican los datasets seleccionados con Python y pandas, y se documentan los problemas de calidad y las acciones de limpieza propuestas.

## Propósito

Identificar las comunas con más propiedad horizontal para decidir dónde enfocar primero Domux, un software de administración de propiedades horizontales que se desarrolla como proyecto de grado.

## Pregunta analítica

> ¿En qué comunas de Medellín se concentran los predios habitacionales en propiedad horizontal y qué proporción representan frente al total de predios habitacionales de cada comuna?

## Datasets

Los archivos CSV **no se publican en el repositorio** por su tamaño (el de predios pesa unos 92 MB). Se deben descargar y guardar en la carpeta `data/raw/` con los nombres indicados.

| Dataset | Archivo en `data/raw/` | Fuente y descarga |
|---|---|---|
| Información Predios | `informacion_predios.csv` | MEData, Alcaldía de Medellín: https://medata.gov.co/node/16594 (enlace "Información Predios (csv)") |
| Información Lotes | `informacion_lotes.csv` | MEData, Alcaldía de Medellín: https://medata.gov.co/node/16593 (enlace "Información Lotes (csv)") |
| Medellín Properties (2023) | `medellin_properties.csv` | Kaggle: https://www.kaggle.com/datasets/cesaregr/medelln-properties (descargar y descomprimir) |
| Precios inmuebles Medellín | `houses_medellin.csv` | Kaggle: https://www.kaggle.com/datasets/valentinafeve/precios-inmuebles-medelln (descargar y descomprimir) |

Los diccionarios de datos de MEData (`Informacion_Predios.json` e `Informacion_Lotes.json`) sí están incluidos en `data/raw/`.

## Estructura del repositorio

```
propiedad-horizontal-en-Medell-n/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   └── raw/                          # CSV descargados (ignorados por Git) y diccionarios JSON
├── notebooks/
│   └── diagnostico_calidad.ipynb     # Carga, inspección y diagnóstico de cada dataset
└── reports/
    └── informe_diagnostico.md        # Informe técnico del diagnóstico
```

## Instrucciones para reproducir el diagnóstico

Requisitos: Python 3.11 o superior y Git.

1. Clonar el repositorio y entrar a la carpeta:
   ```
   git clone https://github.com/MAICOL-ESNEIDER/propiedad-horizontal-en-Medell-n.git
   cd propiedad-horizontal-en-Medell-n
   ```
2. Crear y activar el entorno virtual:
   ```
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1        # Windows PowerShell
   source .venv/bin/activate          # macOS y Linux
   ```
3. Instalar las dependencias:
   ```
   python -m pip install -r requirements.txt
   ```
4. Descargar los cuatro CSV (tabla de *Datasets*) y guardarlos en `data/raw/`.
5. Abrir JupyterLab desde la raíz del proyecto:
   ```
   jupyter lab
   ```
6. Abrir `notebooks/diagnostico_calidad.ipynb` y ejecutar todas las celdas (*Run → Run All Cells*).

## Resultados

Los hallazgos, las acciones de limpieza propuestas y las conclusiones están en [`reports/informe_diagnostico.md`](reports/informe_diagnostico.md).
