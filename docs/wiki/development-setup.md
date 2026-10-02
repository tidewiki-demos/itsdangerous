# Configuración y desarrollo

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Guía para configurar el entorno de desarrollo y entender las herramientas y dependencias del proyecto ItsDangerous.

## Configuración rápida del entorno

### Con Dev Containers (recomendado)

El proyecto incluye configuración para [Dev Containers](https://containers.dev/), que prepara automáticamente el entorno completo. [`.devcontainer/devcontainer.json`](../../.devcontainer/devcontainer.json)

La imagen base es `mcr.microsoft.com/devcontainers/python:3`. Al crear el contenedor, se ejecuta automáticamente [`.devcontainer/on-create-command.sh:1-7`](../../.devcontainer/on-create-command.sh#L1-L7), que:

1. Crea un virtualenv con dependencias actualizadas
2. Instala las dependencias de desarrollo
3. Instala el proyecto en modo editable
4. Configura los hooks de pre-commit

### Configuración manual local

Para configurar localmente sin Dev Containers, sigue estos pasos (basado en [`CONTRIBUTING.rst:98-122`](../../CONTRIBUTING.rst#L98-L122)):

```bash
python3 -m venv env
. env/bin/activate  # En Windows: env\Scripts\activate
pip install -r requirements/dev.txt
pip install -e .
pre-commit install
```

## Estructura de dependencias

Las dependencias están organizadas en archivos `.in` compilados a `.txt` con pip-compile [`requirements/dev.txt:1-6`](../../requirements/dev.txt#L1-L6). Esta estructura separa las preocupaciones:

- **requirements/build.txt** [`requirements/build.txt`](../../requirements/build.txt): Herramientas para construir el paquete (build, packaging, pyproject-hooks)
- **requirements/docs.txt** [`requirements/docs.txt`](../../requirements/docs.txt): Sphinx y temas para generar documentación
- **requirements/tests.txt** [`requirements/tests.txt`](../../requirements/tests.txt): pytest y freezegun para pruebas
- **requirements/typing.txt** [`requirements/typing.txt`](../../requirements/typing.txt): mypy y pyright para análisis de tipos
- **requirements/dev.txt** [`requirements/dev.in:1-5`](../../requirements/dev.in#L1-L5): Todos los anteriores más pre-commit y tox

## Herramientas de desarrollo

### Formateo y linting

El proyecto usa **Ruff** para linting y formateo de código. La configuración [`.pre-commit-config.yaml:2-6`](../../.pre-commit-config.yaml#L2-L6) ejecuta:

- `ruff`: Verifica el código contra reglas B (bugbear), E (pycodestyle error), F (pyflakes), I (isort), UP (pyupgrade) y W (pycodestyle warning) [`pyproject.toml:75-83`](../../pyproject.toml#L75-L83)
- `ruff-format`: Formatea el código automáticamente

Otras verificaciones pre-commit [`.pre-commit-config.yaml:7-14`](../../.pre-commit-config.yaml#L7-L14):

- Detecta conflictos de merge
- Identifica sentencias de debug
- Corrige marcas de orden de byte
- Elimina espacios en blanco finales
- Arregla terminaciones de archivo

### Verificación de tipos

Se utilizan dos verificadores de tipos con configuraciones estrictas:

- **mypy** [`pyproject.toml:57-62`](../../pyproject.toml#L57-L62): python_version 3.8, modo strict
- **pyright** [`pyproject.toml:64-67`](../../pyproject.toml#L64-L67): typeCheckingMode "basic"

Ambos se ejecutan con tox en el ambiente `typing` [`tox.ini:23-28`](../../tox.ini#L23-L28).

### Pruebas

Las pruebas se ejecutan con pytest. Configuración en [`pyproject.toml:44-48`](../../pyproject.toml#L44-L48):

- Ruta de pruebas: `tests`
- Las advertencias se tratan como errores

Cobertura de código: [`pyproject.toml:50-55`](../../pyproject.toml#L50-L55)

```bash
pytest                              # Pruebas básicas
coverage run -m pytest             # Generar reporte de cobertura
coverage html                      # Explorar en htmlcov/index.html
tox                               # Suite completa en múltiples Python
```

### Documentación

La documentación se construye con Sphinx [`CONTRIBUTING.rst:205-215`](../../CONTRIBUTING.rst#L205-L215):

```bash
cd docs
make html
# Abre _build/html/index.html
```

Configuración: [`pyproject.toml:32-39`](../../pyproject.toml#L32-L39) incluye `docs/`, `requirements/`, `tests/` y archivos de cambios.

## Workflow para contribuidores

### Preparación inicial

1. Configura Git con tu nombre y email [`CONTRIBUTING.rst:73-78`](../../CONTRIBUTING.rst#L73-L78)
2. Fork el repositorio
3. Clona localmente: `git clone https://github.com/pallets/itsdangerous`
4. Añade tu fork como remote: `git remote add fork https://github.com/{username}/itsdangerous`

### Rama de trabajo

- **Correcciones de bugs o documentación**: Rama a partir de `origin/2.0.x`
- **Nuevas características**: Rama a partir de `origin/main`

Ver [`CONTRIBUTING.rst:135-150`](../../CONTRIBUTING.rst#L135-L150).

### Antes de enviar un PR

Asegúrate de que:

1. El código esté formateado (pre-commit lo hace automáticamente)
2. Incluyas pruebas que fallan sin tu parche
3. Las pruebas pasen: `pytest`
4. Actualices la documentación relevante (máximo 72 caracteres por línea)
5. Añadas un entry en `CHANGES.rst` y docstrings con `.. versionchanged::`

Ver [`CONTRIBUTING.rst:52-63`](../../CONTRIBUTING.rst#L52-L63).

## Configuración de Tox

Tox ejecuta pruebas en múltiples versiones de Python y ambientes especializados [`tox.ini`](../../tox.ini):

- `py3{13,12,11,10,9,8}`: CPython 3.8 a 3.13
- `pypy310`: PyPy 3.10
- `style`: Verifica formato y linting con pre-commit
- `typing`: Ejecuta mypy y pyright
- `docs`: Construye la documentación
- `update-actions`: Actualiza las acciones de GitHub con gha-update
- `update-pre_commit`: Actualiza los hooks de pre-commit
- `update-requirements`: Recompila los archivos de requisitos con pip-compile

Comando: `tox` (o `tox -e typing` para un ambiente específico).

## Decisions

- **Python 3.13 en la matriz de pruebas** (commit 2da4fabb2213): Se añadió Python 3.13 a la lista de versiones testeadas en `tox.ini`, expandiendo de `py3{12,11,10,9,8}` a `py3{13,12,11,10,9,8}`.

- **Cambios en automatización de actualizaciones** (commits ad4174f458fc y 0df2193cd68d): Se removió la configuración `ci.autoupdate_schedule` de `.pre-commit-config.yaml` y se separó el trabajo de actualización en tox en tres ambientes especializados (`update-actions`, `update-pre_commit`, `update-requirements`). Esto reemplaza el modelo anterior donde pre-commit autoupdate se ejecutaba como parte de `update-requirements` y se utiliza ahora `gha-update` para actualizar las acciones de GitHub.

## Configuración del proyecto

El proyecto usa:

- **Build system**: flit [`pyproject.toml:25-27`](../../pyproject.toml#L25-L27)
- **Python mínimo**: 3.8 [`pyproject.toml:16`](../../pyproject.toml#L16)
- **Licencia**: BSD [`pyproject.toml:6`](../../pyproject.toml#L6)
- **Nombre del módulo**: `itsdangerous` [`pyproject.toml:29-30`](../../pyproject.toml#L29-L30)

Ver [Sistema de documentación](documentation.md) para detalles sobre la generación de docs. Ver [Integración continua y publicación](ci-and-deployment.md) para CI/CD.
