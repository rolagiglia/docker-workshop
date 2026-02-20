# Pipeline de Ingestión de Datos de Taxis de NYC

Este proyecto es un pipeline de datos diseñado para ingestar y almacenar datos de taxis de Nueva York en una base de datos PostgreSQL. Utiliza contenedores Docker para facilitar la ejecución y el despliegue.

## Descripción General

El pipeline consta de varios componentes que trabajan juntos para descargar, procesar e insertar datos de taxis en una base de datos PostgreSQL. Los datos incluyen información de viajes de taxis amarillos y datos de zonas de servicio.

## Componentes Principales

### 1. `ingest_data.py`
Este script es responsable de la ingestión de datos de viajes de taxis amarillos de Nueva York.

**Funcionalidades:**
- Descarga datos CSV comprimidos desde el repositorio de DataTalksClub en GitHub
- Procesa los datos en chunks para manejar archivos grandes eficientemente
- Inserta los datos en una tabla PostgreSQL llamada `yellow_taxi_data`
- Utiliza tipos de datos específicos para optimizar el almacenamiento
- Maneja fechas de pickup y dropoff

**Parámetros de línea de comandos:**
- `--pg-user`: Usuario de PostgreSQL (default: root)
- `--pg-pass`: Contraseña de PostgreSQL (default: root)
- `--pg-host`: Host de PostgreSQL (default: localhost)
- `--pg-port`: Puerto de PostgreSQL (default: 5432)
- `--pg-db`: Nombre de la base de datos (default: ny_taxi)
- `--target-table`: Nombre de la tabla destino (default: yellow_taxi_data)

**Ejemplo de uso:**
```bash
python ingest_data.py --pg-user root --pg-pass root --pg-host localhost --pg-port 5432 --pg-db ny_taxi
```

### 2. `ingest_zones_data.py`
Este script ingesta datos de zonas de taxis de Nueva York.

**Funcionalidades:**
- Descarga el archivo `taxi_zone_lookup.csv` desde GitHub
- Procesa los datos en chunks
- Inserta los datos en la tabla `taxi_service_zones`
- Incluye información sobre boroughs, zonas y tipos de servicio

**Configuración:**
Los parámetros de conexión están hardcodeados en el script:
- Usuario: root
- Contraseña: root
- Host: localhost
- Puerto: 5432
- Base de datos: ny_taxi
- Tabla destino: taxi_service_zones

**Ejemplo de uso:**
```bash
python ingest_zones_data.py
```

### 3. `Dockerfile`
El Dockerfile define la imagen de contenedor para ejecutar el script de ingestión.

**Características:**
- Basado en Python 3.13.10-slim
- Utiliza `uv` para gestión de dependencias (más rápido que pip)
- Instala dependencias desde `pyproject.toml` y `uv.lock`
- Configura un entorno virtual
- Establece `ingest_data.py` como punto de entrada

**Construcción de la imagen:**
```bash
docker build -t taxi-ingest .
```

### 4. `docker-compose.yaml`
Archivo de composición de Docker que orquesta los servicios necesarios.

**Servicios:**
- **pgdatabase**: Instancia de PostgreSQL 18
  - Usuario: root
  - Contraseña: root
  - Base de datos: ny_taxi
  - Puerto expuesto: 5432
  - Volumen persistente para datos

- **pgadmin**: Interfaz web pgAdmin4 para gestión de PostgreSQL
  - Email: admin@admin.com
  - Contraseña: root
  - Puerto expuesto: 8085
  - Volumen persistente para configuración

**Volúmenes:**
- `ny_taxi_postgres_data`: Para persistir datos de PostgreSQL
- `pgadmin_data`: Para persistir configuración de pgAdmin

## Dependencias

Las dependencias del proyecto están definidas en `pyproject.toml`:

**Dependencias principales:**
- pandas >= 3.0.0
- psycopg[binary] >= 3.1.0
- pyarrow >= 23.0.0
- sqlalchemy >= 2.0.46
- tqdm >= 4.67.3

**Dependencias de desarrollo:**
- jupyter >= 1.1.1
- pgcli >= 4.4.0

## Instalación y Configuración

### Prerrequisitos
- Docker y Docker Compose instalados
- Git (opcional, para clonar el repositorio)

### Pasos de Instalación

1. **Clonar el repositorio:**
   ```bash
   git clone <url-del-repositorio>
   cd pipeline
   ```

2. **Levantar los servicios con Docker Compose:**
   ```bash
   docker-compose up -d
   ```

   Esto iniciará PostgreSQL y pgAdmin en segundo plano.

3. **Verificar que los servicios estén corriendo:**
   ```bash
   docker-compose ps
   ```

4. **Acceder a pgAdmin:**
   - Abrir navegador en http://localhost:8085
   - Usuario: admin@admin.com
   - Contraseña: root

## Uso

### Ejecutar la Ingestión de Datos

**Opción 1: Usando Docker (recomendado)**
```bash
docker run --network pipeline_default \
  -e PG_USER=root \
  -e PG_PASS=root \
  -e PG_HOST=pgdatabase \
  -e PG_PORT=5432 \
  -e PG_DB=ny_taxi \
  taxi-ingest
```

**Opción 2: Ejecutar localmente**
Asegúrate de tener Python 3.13+ y las dependencias instaladas:
```bash
pip install -e .
python ingest_data.py
```

### Ejecutar la Ingestión de Zonas
```bash
python ingest_zones_data.py
```

## Estructura de Datos

### Tabla `yellow_taxi_data`
Contiene datos de viajes de taxis amarillos con campos como:
- VendorID
- tpep_pickup_datetime
- tpep_dropoff_datetime
- passenger_count
- trip_distance
- PULocationID, DOLocationID
- fare_amount, tip_amount, total_amount
- etc.

### Tabla `taxi_service_zones`
Contiene información de zonas:
- LocationID
- Borough
- Zone
- service_zone

## Desarrollo

Para desarrollo local:

1. Instalar dependencias de desarrollo:
   ```bash
   uv sync --group dev
   ```

2. Activar el entorno virtual:
   ```bash
   source .venv/bin/activate
   ```

3. Ejecutar Jupyter Notebook:
   ```bash
   jupyter notebook
   ```

## Contribución

1. Fork el proyecto
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

## Licencia

Este proyecto está bajo la Licencia MIT. Ver el archivo `LICENSE` para más detalles.

## Contacto

Para preguntas o soporte, por favor abre un issue en el repositorio de GitHub.