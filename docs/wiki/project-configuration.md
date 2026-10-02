# Configuración del Proyecto

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Define la construcción, dependencias y metadatos del paquete itsdangerous. La configuración se centraliza en `pyproject.toml` siguiendo el estándar PEP 517/518, complementada con archivos de requisitos para distintos propósitos.

## Metadatos del Paquete

[`pyproject.toml:1-15`](../../pyproject.toml#L1-L15)

El proyecto se identifica como **itsdangerous** versión 2.2.0, mantenido por Pallets. Su propósito es "pasar datos de forma segura a entornos no confiables y recuperarlos". Está clasificado como "Production/Stable" y está completamente tipado.

- **Licencia**: BSD (archivo `LICENSE.txt`)
- **Requisitos mínimos**: Python 3.8 o superior [`pyproject.toml:16`](../../pyproject.toml#L16)
- **Lectores objetivo**: Desarrolladores

## URLs del Proyecto

[`pyproject.toml:18-23`](../../pyproject.toml#L18-L23)

Se define la documentación oficial en https://itsdangerous.palletsprojects.com/, junto con enlaces a:
- Donaciones y financiamiento
- Registro de cambios
- Repositorio de código fuente en GitHub
- Comunidad en Discord

## Sistema de Construcción

[`pyproject.toml:25-27`](../../pyproject.toml#L25-L27)

El proyecto utiliza **flit** como sistema de construcción (`flit_core<4`), un generador de paquetes minimalista que infiere metadatos de la estructura del código.

[`pyproject.toml:29-31`](../../pyproject.toml#L29-L31)

El módulo principal se llama `itsdangerous` en lugar de usar una estructura `src/`.

## Contenido del Distributable

[`pyproject.toml:32-42`](../../pyproject.toml#L32-L42)

Los archivos distribuibles incluyen:
- `docs/` — documentación fuente
- `requirements/` — especificaciones de dependencias
- `tests/` — suite de pruebas
- `CHANGES.rst` — historial de cambios
- `tox.ini` — configuración de pruebas

Se excluye `docs/_build/` para evitar artefactos compilados.

## Configuración de Pruebas

[`pyproject.toml:44-48`](../../pyproject.toml#L44-L48)

Las pruebas se ejecutan desde el directorio `tests/`. Se configura pytest para tratar advertencias como errores (`filterwarnings = ["error"]`), forzando que cualquier warning rompa la ejecución.

[`pyproject.toml:50-55`](../../pyproject.toml#L50-L55)

La cobertura se mide con ramificación habilitada, incluyendo tanto el código fuente como los tests.

## Validación de Tipos

[`pyproject.toml:57-62`](../../pyproject.toml#L57-L62)

**mypy** se ejecuta en modo `strict` contra Python 3.8, verificando el código en `src/itsdangerous`.

[`pyproject.toml:64-67`](../../pyproject.toml#L64-L67)

**pyright** usa verificación de tipo en modo `basic`, también dirigido a Python 3.8.

## Calidad de Código

[`pyproject.toml:69-87`](../../pyproject.toml#L69-L87)

**ruff** se configura como linter principal con las siguientes reglas activas:
- `B` — flake8-bugbear (detección de errores comunes)
- `E`, `W` — pycodestyle (errores y advertencias de estilo)
- `F` — pyflakes (errores lógicos)
- `I` — isort (ordenamiento de importaciones)
- `UP` — pyupgrade (modernización de sintaxis)

Se habilita corrección automática (`fix = true`). Las importaciones se fuerzan a línea única.

## Dependencias por Propósito

Las dependencias se organizan en archivos de requisitos compilados mediante `pip-compile`:

### Construcción
[`requirements/build.txt:1-12`](../../requirements/build.txt#L1-L12)

Solo `build==1.2.1` y sus dependencias (`packaging`, `pyproject-hooks`).

### Documentación
[`requirements/docs.txt:1-57`](../../requirements/docs.txt#L1-L57)

Stack basado en **Sphinx** con tema de Pallets: `sphinx`, `pallets-sphinx-themes`, `sphinxcontrib-log-cabinet`.

### Pruebas
[`requirements/tests.txt:1-20`](../../requirements/tests.txt#L1-L20)

- `pytest==8.2.0` — ejecutor de pruebas
- `freezegun==1.5.0` — simula tiempo en tests

### Validación de Tipos
[`requirements/typing.txt:1-24`](../../requirements/typing.txt#L1-L24)

- `mypy==1.10.0` — verificador estático
- `pyright==1.1.360` — verificador alternativo de Microsoft
- `pytest==8.2.0` — incluido para pruebas de tipo

### Desarrollo Completo
[`requirements/dev.txt:1-177`](../../requirements/dev.txt#L1-L177)

Incluye todas las categorías anteriores más:
- `pre-commit==3.7.0` — automatización de hooks git
- `tox==4.15.0` — gestor de entornos virtuales para pruebas matriciales

## Decisiones

**Strict mode en mypy**: Se elige verificación de tipo en modo `strict` para máxima seguridad en una librería de criptografía y serialización. Esto alinea con el propósito crítico del paquete (ver [Conceptos Fundamentales](concepts.md)).

**flit como build backend**: Se utiliza flit en lugar de setuptools por su simplificidad y mejor integración con PEP 621. Infiere la versión y metadatos del módulo directamente.

**Requisitos compilados con pip-compile**: Los archivos `.txt` son generados y versionados, no editados manualmente. Esto asegura reproducibilidad de builds y actualizaciones controladas.
