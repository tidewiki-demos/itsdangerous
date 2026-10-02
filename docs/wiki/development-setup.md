# Configuración y desarrollo

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Guía para configurar el entorno de desarrollo y entender las herramientas y dependencias del proyecto ItsDangerous.

## Configuración rápida del entorno

### Con Dev Containers (recomendado)

El proyecto incluye configuración para [Dev Containers](https://containers.dev/), que prepara automáticamente el entorno completo. [`.devcontainer/devcontainer.json`](../../.devcontainer/devcontainer.json)

La imagen base es `mcr.microsoft.com/devcontainers/python:3`. Al crear el contenedor, se ejecuta automáticamente [`.devcontainer/on-create-command.sh:1-17`](../../.devcontainer/on-create-command.sh#L1-L17), que:

1. Instala `uv` si no está disponible
2. Crea un virtualenv con `uv sync` e instala las dependencias
3. Configura los hooks de pre-commit

### Configuración manual local

Para configurar localmente sin Dev Containers, instala `uv` (herramienta para gestionar dependencias y entornos Python) y luego:

```bash
uv sync
pre-commit install --install-hooks
```

## Estructura de dependencias

Las dependencias se organizan como grupos de dependencias en `pyproject.toml` [`pyproject.toml:25-54`](../../pyproject.toml#L25-L54) y se gestionan con `uv`:

- **dev**: Herramientas para desarrollo (ruff, tox, tox-uv)
- **docs**: Sphinx y temas para generar documentación
- **docs-auto**: sphinx-autobuild para desarrollo continuo de docs
- **pre-commit**: pre-commit y pre-commit-uv para hooks
- **tests**: pytest y freezegun para pruebas
- **typing**: mypy y pyright para análisis de tipos
- **gha-update**: gha-update para actualizar acciones de GitHub (requiere Python 3.12+)

El archivo `uv.lock` [`pyproject.toml:75-76`](../../pyproject.toml#L75-L76) proporciona dependencias reproducibles.

## Herramientas de desarrollo

### Formateo y linting

El proyecto usa **Ruff** para linting y formateo de código. La configuración [`.pre-commit-config.yaml:2-6`](../../.pre-commit-config.yaml#L2-L6) ejecuta:

- `ruff`: Verifica el código contra reglas B (bugbear), E (pycodestyle error), F (pyflakes), I (isort), UP (pyupgrade) y W (pycodestyle warning) [`pyproject.toml:116-127`](../../pyproject.toml#L116-L127)
- `ruff-format`: Formatea el código automáticamente

Otras verificaciones pre-commit [`.pre-commit-config.yaml:7-18`](../../.pre-commit-config.yaml#L7-L18):

- Hook `uv-lock`: Verifica la consistencia del archivo `uv.lock`
- Detecta conflictos de merge
- Identifica sentencias de debug
- Corrige marcas de orden de byte
- Elimina espacios en blanco finales
- Arregla terminaciones de archivo

### Verificación de tipos

Se utilizan dos verificadores de tipos con configuraciones estrictas:

- **mypy** [`pyproject.toml:98-103`](../../pyproject.toml#L98-L103): python_version 3.10, modo strict
- **pyright** [`pyproject.toml:105-108`](../../pyproject.toml#L105-L108): pythonVersion 3.10, typeCheckingMode "standard"

Ambos se ejecutan con tox en el ambiente `typing` [`pyproject.toml:166-173`](../../pyproject.toml#L166-L173).

### Pruebas

Las pruebas se ejecutan con pytest. Configuración en [`pyproject.toml:78-82`](../../pyproject.toml#L78-L82):

- Ruta de pruebas: `tests`
- Las advertencias se tratan como errores

Cobertura de código: [`pyproject.toml:84-96`](../../pyproject.toml#L84-L96)

```bash
pytest                              # Pruebas básicas
coverage run -m pytest             # Generar reporte de cobertura
coverage html                      # Explorar en htmlcov/index.html
tox                               # Suite completa en múltiples Python
```

### Documentación

La documentación se construye con Sphinx. Comandos disponibles:

```bash
# Construir documentación una sola vez
tox -e docs

# Reconstruir continuamente con servidor local
tox -e docs-auto
```

Configuración en tox: [`pyproject.toml:175-183`](../../pyproject.toml#L175-L183).

## Configuración de Tox

Tox ejecuta pruebas en múltiples versiones de Python y ambientes especializados [`pyproject.toml:138-205`](../../pyproject.toml#L138-L205):

- `py3.13`, `py3.12`, `py3.11`, `py3.10`: CPython 3.10 a 3.13
- `pypy311`: PyPy 3.11
- `style`: Verifica formato y linting con pre-commit
- `typing`: Ejecuta mypy y pyright
- `docs`: Construye la documentación
- `docs-auto`: Reconstruye documentación continuamente
- `update-actions`: Actualiza los pins de las acciones de GitHub con gha-update
- `update-pre_commit`: Actualiza los hooks de pre-commit
- `update-requirements`: Recompila el archivo `uv.lock` con uv

Comando: `tox` (o `tox -e typing` para un ambiente específico).

## Decisions

- **Migración a uv y cambio de Python mínimo a 3.10** (commits 91952b90cd5c y 7edfa12ed562): Se reemplazó el sistema de requisitos basado en pip-compile con `uv` para gestión de dependencias, eliminando archivos `requirements/*.txt` y `requirements/*.in`. Al mismo tiempo, se incrementó el soporte mínimo de Python de 3.8 a 3.10, y se actualizó la matriz de pruebas de `py3{13,12,11,10,9,8}` a `py3{13,12,11,10}` más `pypy311`. Los verificadores de tipos (mypy y pyright) también se actualizaron a pythonVersion 3.10, y pyright cambió de typeCheckingMode "basic" a "standard".

- **Cambios en automatización de actualizaciones** (commits anteriores ad4174f458fc y 0df2193cd68d, ahora consolidados): Los ambientes de tox para actualización se redefinieron: `update-actions` usa gha-update, `update-pre_commit` usa pre-commit autoupdate, y el nuevo `update-requirements` reemplaza la lógica anterior de pip-compile con `uv lock`.

## Configuración del proyecto

El proyecto usa:

- **Build system**: flit [`pyproject.toml:56-58`](../../pyproject.toml#L56-L58)
- **Python mínimo**: 3.10 [`pyproject.toml:16`](../../pyproject.toml#L16)
- **Licencia**: BSD-3-Clause [`pyproject.toml:6`](../../pyproject.toml#L6)
- **Nombre del módulo**: `itsdangerous` [`pyproject.toml:60-61`](../../pyproject.toml#L60-L61)

Ver [Sistema de documentación](documentation.md) para detalles sobre la generación de docs. Ver [Integración continua y publicación](ci-and-deployment.md) para CI/CD.
