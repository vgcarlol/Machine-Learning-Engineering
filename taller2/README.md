# Taller 2 - Ambientes virtuales

Carlos Valladares - 221164

## 1. Documentación

- Python venv: https://docs.python.org/3/library/venv.html
- Tutorial de ambientes virtuales: https://docs.python.org/3/tutorial/venv.html
- VS Code: https://code.visualstudio.com/docs/python/environments
- Conda: https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html

## 2. Ambiente con venv

El archivo de requisitos por convención se llama `requirements.txt`.

```bash
cd taller2
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Para generar el archivo desde un ambiente ya instalado se puede usar `pip freeze > requirements.txt`, pero ese comando incluye todas las dependencias indirectas. Por eso aquí solo dejé los paquetes principales con su versión.

## 3. Requisitos del pipeline

`pipeline_sklearn/` es el pipeline de la actividad 1 (preparación de datos y PCA). Su archivo de requisitos está en `pipeline_sklearn/requirements.txt`.

Reto: para que el paquete use ese archivo hice dos cambios.

- En `pyproject.toml` las dependencias pasaron a ser dinámicas y se leen de `requirements.txt`:

  ```toml
  [project]
  dynamic = ["dependencies"]

  [tool.setuptools.dynamic]
  dependencies = { file = ["requirements.txt"] }
  ```

- Agregué `MANIFEST.in` con `include requirements.txt`. Sin esto el archivo no se copia al `.tar.gz` y la instalación desde ese paquete falla.

Para comprobarlo:

```bash
pip install build
python -m build pipeline_sklearn
```

El `.tar.gz` generado en `pipeline_sklearn/dist/` contiene `requirements.txt`, y el metadata del `.whl` lista las dependencias.

## 4. Ambiente con Conda

```bash
conda env create -f environment.yml
conda activate taller2-ml
```

Conda no usa `requirements.txt` sino `environment.yml`. Las diferencias principales:

- Incluye el nombre del ambiente y la versión de Python, no solo los paquetes.
- Se pueden definir canales (`conda-forge`).
- Permite una sección `pip:` para paquetes que no están en conda. Aquí la usé para instalar el pipeline local.

Para exportar un ambiente existente: `conda env export --from-history > environment.yml`.

## 5. Comparación con uv

Referencia: https://www.datacamp.com/tutorial/python-uv

| Aspecto | venv + pip | uv |
|---|---|---|
| Qué es | Módulo estándar de Python + instalador de paquetes | Gestor de paquetes y proyectos escrito en Rust |
| Crear ambiente | python -m venv .venv | uv venv |
| Instalar paquetes | pip install -r requirements.txt | uv pip install -r requirements.txt o uv add paquete |
| Archivo de dependencias | requirements.txt | pyproject.toml + uv.lock |
| Lock file | No tiene; se aproxima con pip freeze | uv.lock se genera automáticamente |
| Velocidad | Normal | Mucho más rápido (resolución e instalación en paralelo, caché) |
| Versiones de Python | No las instala; usa la del sistema | uv python install descarga y gestiona versiones |
| Compatibilidad | Estándar, viene con Python | Acepta requirements.txt y comandos tipo pip |


## 6. Comparación con Poetry

Referencia: https://www.datacamp.com/tutorial/python-poetry

| Aspecto | venv + pip | Poetry |
|---|---|---|
| Qué es | Módulo estándar de Python + instalador de paquetes | Herramienta de gestión de dependencias y empaquetado |
| Crear ambiente | python -m venv .venv (manual) | Automático con poetry install |
| Agregar paquete | pip install paquete y editar requirements.txt | poetry add paquete (actualiza pyproject.toml) |
| Archivo de dependencias | requirements.txt | pyproject.toml + poetry.lock |
| Grupos de dependencias | Archivos separados (ej. requirements-dev.txt) | Grupos en pyproject.toml (--group dev) |
| Resolución de conflictos | Básica, reporta conflictos al instalar | Resuelve todo el árbol antes de instalar |
| Empaquetar y publicar | Requiere setuptools, build y twine | poetry build y poetry publish |
| Versiones de Python | No las instala | No las instala; usa las existentes (pyenv) |


## Capturas

Las capturas están en `capturas/`.

## Conclusiones

- Los ambientes virtuales permiten que cada proyecto tenga sus propias versiones de paquetes. En Machine Learning esto importa porque un cambio de versión en scikit-learn o pandas puede cambiar resultados o romper un modelo guardado.
- El `requirements.txt` con versiones fijas hace que el entrenamiento se pueda repetir en otra computadora o en un servidor y obtener el mismo ambiente con un solo comando.
- Conda es útil cuando el proyecto necesita dependencias que no son de Python, como librerías para GPU, y su `environment.yml` guarda también la versión de Python.
- Incluir el archivo de requisitos dentro del paquete del pipeline evita que las dependencias queden repetidas en dos lugares y facilita pasar el pipeline a producción o a un contenedor.
