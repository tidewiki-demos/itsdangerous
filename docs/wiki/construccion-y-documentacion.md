# Construcción y Documentación

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Esta página describe cómo se configura el proyecto, se construye la documentación con Sphinx y se publica el paquete.

## Configuración del Proyecto

El proyecto utiliza [`pyproject.toml:1-74`](../../pyproject.toml#L1-L74) `pyproject.toml` como archivo de configuración principal, que sigue el estándar moderno de Python. La configuración incluye:

- **Metadatos del proyecto**: nombre (`itsdangerous`), versión (`2.3.0.dev`), descripción y licencia BSD-3-Clause [`pyproject.toml:1-16`](../../pyproject.toml#L1-L16)
- **URLs del proyecto**: documentación, código fuente, chat y enlaces de donación [`pyproject.toml:18-23`](../../pyproject.toml#L18-L23)
- **Requisitos Python**: versión mínima 3.10 [`pyproject.toml:16`](../../pyproject.toml#L16)
- **Sistema de construcción**: usa `flit_core` como backend [`pyproject.toml:56-58`](../../pyproject.toml#L56-L58)

### Grupos de Dependencias

El proyecto organiza las dependencias en grupos temáticos [`pyproject.toml:25-54`](../../pyproject.toml#L25-L54):

- `dev`: herramientas de desarrollo (ruff, tox)
- `docs`: dependencias para construcción de documentación (Sphinx, temas Pallets)
- `docs-auto`: reconstrucción automática de documentación
- `tests`: framework de pruebas (pytest, freezegun)
- `typing`: verificadores de tipos estáticos (mypy, pyright)
- `pre-commit`: hooks de pre-commit

## Construcción de Documentación con Sphinx

La documentación se construye con Sphinx y utiliza temas personalizados de Pallets.

### Configuración de Sphinx

[`docs/conf.py:1-55`](../../docs/conf.py#L1-L55) contiene la configuración principal:

- **Tema**: usa el tema `flask` de `pallets_sphinx_themes` [`docs/conf.py:35`](../../docs/conf.py#L35)
- **Extensiones habilitadas**: [`docs/conf.py:14-20`](../../docs/conf.py#L14-L20)
  - `sphinx.ext.autodoc`: genera documentación automáticamente desde docstrings
  - `sphinx.ext.extlinks`: crea enlaces externos acortados
  - `sphinx.ext.intersphinx`: vincula a documentación externa (Python)
  - `sphinxcontrib.log_cabinet`: gestiona registros de cambios
  - `pallets_sphinx_themes`: proporciona el tema y funcionalidades

- **Opciones de autodoc**: [`docs/conf.py:21-24`](../../docs/conf.py#L21-L24)
  - `autoclass_content = "both"`: incluye docstrings de clase e `__init__`
  - `autodoc_member_order = "bysource"`: mantiene el orden del código fuente
  - `autodoc_typehints = "description"`: muestra anotaciones de tipo en descripciones
  - `autodoc_preserve_defaults = True`: preserva valores por defecto

### Estructura de la Documentación

La documentación raíz está en el directorio `docs/` y se organiza así:

- [`docs/index.rst:1-63`](../../docs/index.rst#L1-L63) `index.rst`: página principal con tabla de contenidos que incluye conceptos, interfaces de firma/serialización, excepciones, soporte de timestamps, serialización URL-segura, codificación, licencia y cambios
- [`docs/concepts.rst:1-154`](../../docs/concepts.rst#L1-L154) `concepts.rst`: explica conceptos fundamentales como la diferencia entre Signer y Serializer, gestión de claves secretas, salts y rotación de claves
- `signer.rst`, `serializer.rst`, `timed.rst`, `url_safe.rst`: documentación de interfaces
- `encoding.rst`, `exceptions.rst`: utilidades y excepciones
- `license.rst`, `changes.rst`: licencia y changelog

### Construcción Manual

Se proporciona un Makefile [`docs/Makefile:1-18`](../../docs/Makefile#L1-L18) que automatiza la construcción:

```bash
make html          # Construye documentación HTML
make dirhtml        # Construye documentación en formato dirhtml
```

En Windows, usar `make.bat` [`docs/make.bat:1-35`](../../docs/make.bat#L1-L35).

### Construcción con Tox

Para construcción automática durante desarrollo:

[`pyproject.toml:180-183`](../../pyproject.toml#L180-L183)

```bash
tox run -e docs-auto
```

Este comando reconstruye la documentación continuamente y sirve un servidor local. Usa Sphinx autobuild con la opción `-W` para tratar advertencias como errores.

Para construcción estándar:

[`pyproject.toml:175-178`](../../pyproject.toml#L175-L178)

```bash
tox run -e docs
```

Construye documentación en formato `dirhtml` en el directorio `docs/_build/dirhtml/`.

## Publicación del Paquete

La publicación se gestiona a través de herramientas estándar de Python:

### Sistema de Construcción

El proyecto usa `flit_core` [`pyproject.toml:56-58`](../../pyproject.toml#L56-L58), que lee metadatos de `pyproject.toml` y construye el paquete automáticamente. [`pyproject.toml:60-70`](../../pyproject.toml#L60-L70) define qué archivos incluir en la distribución:

- Incluye: `docs/`, `examples/`, `tests/`, `CHANGES.rst`, `uv.lock`
- Excluye: `docs/_build/` (artefactos de construcción)

### Distribución

El módulo raíz se especifica en [`pyproject.toml:60-61`](../../pyproject.toml#L60-L61):

```toml
[tool.flit.module]
name = "itsdangerous"
```

El proyecto se publica en PyPI. Los URLs de referencia están en [`pyproject.toml:18-23`](../../pyproject.toml#L18-L23), incluyendo la página de releases en PyPI.

## Control de Calidad

Antes de publicar, el proyecto ejecuta varios controles:

### Verificación Estática

[`pyproject.toml:98-108`](../../pyproject.toml#L98-L108) configura verificadores de tipos:

- `mypy` y `pyright` con modo estricto (`strict = true`)
- Versión Python objetivo: 3.10

Ejecutar verificación:

```bash
tox run -e typing
```

### Análisis de Código

[`pyproject.toml:110-131`](../../pyproject.toml#L110-L131) configura `ruff` para linting:

- Reglas: bugbear, errores de estilo, pyflakes, isort, pyupgrade
- Modo corrección automática habilitado

Ejecutar:

```bash
tox run -e style
```

### Pruebas

[`pyproject.toml:78-82`](../../pyproject.toml#L78-L82) configura pytest para ejecutarse en el directorio `tests/` tratando advertencias como errores.

Ejecutar en múltiples versiones Python (3.10, 3.11, 3.12, 3.13, PyPy 3.11):

```bash
tox run
```

## Licencia

El proyecto está bajo licencia BSD-3-Clause [`LICENSE.txt:1-28`](../../LICENSE.txt#L1-L28), incluida en todos los paquetes publicados.
