# Dagster & dbt

Pipeline de datos sobre los viajes en taxi de Nueva York, construido con Dagster, dbt y DuckDB. Es mi práctica de los cursos **Dagster Essentials** y **Dagster & dbt** de Dagster University.

## Qué hace el pipeline

**1. Ingesta (Dagster)**
- `taxi_trips_file`: descarga los viajes de cada mes (particionado mensual).
- `taxi_zones_file`: descarga el catálogo de zonas de NYC.
- `taxi_trips` y `taxi_zones`: cargan esos archivos como tablas raw en DuckDB.

**2. Transformación (dbt)**
- `stg_trips` y `stg_zones`: limpian y estandarizan las tablas raw.
- `location_metrics`: métricas de viajes por zona.
- `daily_metrics`: métricas diarias de viajes (modelo incremental).

**3. Análisis (Dagster)**
- `trips_by_week`: viajes agregados por semana (partición semanal).
- `manhattan_stats`: cantidad de viajes por zona de Manhattan.
- `manhattan_map`: mapa de esos viajes por zona.
- `airport_trips`: gráfico de los destinos de los viajes que salen del aeropuerto, construido sobre el modelo de dbt `location_metrics`.
- `adhoc_request`: reportes bajo demanda, disparados por un sensor.

## Conceptos aplicados

- **Particiones:** mensuales en la ingesta de viajes, semanales en `trips_by_week` y diarias en el modelo incremental de dbt.
- **Jobs, sensors y schedules:** ejecución automática por horario o ante un pedido.
- **Integración con dbt:** los modelos se cargan como assets con `@dbt_assets`; los incrementales tienen su propia definición con `partitions_def`.
- **Resources:** conexión a DuckDB y ejecución de dbt (`DbtCliResource`).

## Stack

Dagster · dbt · DuckDB · Python

## Requisitos

- Python 3.10 a 3.13
- Git
- En Windows, no clonar el proyecto dentro de OneDrive: puede bloquear archivos de dbt y trabar el arranque.
- Se recomienda tener al menos 8 GB de RAM. Si la máquina tiene poca memoria, materializar de a pocos meses y cerrar programas pesados.

## Cómo levantar el proyecto localmente

### 1. Clonar el repositorio

```bash
git clone https://github.com/agusyapura/dagster-dbt.git
cd dagster-dbt
```

### 2. Crear y activar un entorno virtual

**Windows (PowerShell):**

```powershell
python -m venv venv
venv\Scripts\Activate
```

**Linux / macOS:**

```bash
python3 -m venv venv
source venv/bin/activate
```

### 3. Instalar las dependencias

```bash
pip install -e .
```

### 4. Variables de entorno

El archivo `.env` ya está incluido en el repo. Solo define la ruta de la base de DuckDB (`DUCKDB_DATABASE=data/staging/data.duckdb`) y no contiene claves ni datos sensibles. No hace falta crearlo ni modificarlo.

### 5. Iniciar la interfaz de Dagster

Ejecutar siempre desde la raíz del proyecto:

```bash
dagster dev
```

Una vez que levante, abrí el navegador en http://localhost:3000 para ver el linaje y materializar los assets.

### Alternativa con uv

Si usás [uv](https://docs.astral.sh/uv/), podés saltear los pasos 2 y 3:

```bash
uv sync
uv run dagster dev
```

## Probar el sensor

Crear un archivo JSON con nombre único dentro de `data/requests/`:

```json
{"start_date": "2023-01-08", "end_date": "2023-01-22", "borough": "Queens"}
```

Activar `adhoc_request_sensor` en la pestaña **Automation**. En unos 30 segundos lanza el run y genera el reporte. Usar fechas de meses ya materializados en `taxi_trips`, y un nombre de archivo distinto para cada pedido.

## Sobre este proyecto

Proyecto base de los cursos de [Dagster University](https://courses.dagster.io/). La carpeta `completed/` incluye el código terminado de las distintas etapas del curso **Dagster & dbt**.
