# Flujo de Integración Continua

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

El repositorio utiliza workflows de GitHub Actions para automatizar pruebas, verificación de código, construcción y publicación. Estos workflows se ejecutan automáticamente en eventos específicos como push, pull request y creación de tags.

## Workflows

### pre-commit

[`.github/workflows/pre-commit.yaml`](../../.github/workflows/pre-commit.yaml)

Se ejecuta en cada push a `main` o `stable`, y en pull requests. Valida el código mediante pre-commit hooks antes de que se integre.

El workflow:
1. Descarga el repositorio
2. Configura `uv` (gestor de paquetes Python) y Python
3. Cachea configuraciones de pre-commit
4. Ejecuta `uv run --locked --group pre-commit pre-commit run --show-diff-on-failure --color=always --all-files`
5. Ejecuta verificaciones adicionales de pre-commit-ci si no se ha cancelado

Los hooks pre-commit incluyen [`.pre-commit-config.yaml`](../../.pre-commit-config.yaml):
- **ruff**: linter y formateador de código Python
- **uv-lock**: verifica que el archivo de lock esté actualizado
- **pre-commit-hooks**: verificaciones genéricas (conflictos de merge, statements de debug, BOM, espacios finales, saltos de línea)

### tests

[`.github/workflows/tests.yaml`](../../.github/workflows/tests.yaml)

Se ejecuta en push a `main` o `stable` y en pull requests, pero ignora cambios solo en documentación. Ejecuta pruebas unitarias en múltiples configuraciones:

**Job tests**: ejecuta la suite de pruebas usando tox en una matriz que incluye:
- Python 3.13, 3.12, 3.11, 3.10
- PyPy 3.11
- Sistemas operativos: Ubuntu (por defecto), Windows, macOS

Cada configuración ejecuta `uv run --locked tox run` con el entorno correspondiente.

**Job typing**: valida tipos estáticos con mypy en Python (versión del proyecto). Cachea resultados en `./.mypy_cache`.

### publish

[`.github/workflows/publish.yaml`](../../.github/workflows/publish.yaml)

Se ejecuta automáticamente cuando se crea un tag (`push: tags: ['*']`). Orquesta la construcción y publicación del paquete:

**Job build**:
1. Descarga el código
2. Configura `uv` y Python
3. Establece `SOURCE_DATE_EPOCH` para reproducibilidad
4. Ejecuta `uv build` (genera distribuciones en `./dist`)
5. Carga los artefactos

**Job create-release**: depende de `build`
- Descarga los artefactos
- Crea una release en GitHub en modo borrador con los archivos compilados
- Requiere permisos de escritura en contenidos

**Job publish-pypi**: depende de `build`
- Descarga los artefactos
- Publica el paquete en PyPI usando `pypa/gh-action-pypi-publish`
- Usa autenticación OIDC (sin credenciales almacenadas)
- La URL de confirmación apunta a `https://pypi.org/project/itsdangerous/{version}`

### lock

[`.github/workflows/lock.yaml`](../../.github/workflows/lock.yaml)

Se ejecuta diariamente a las 00:00 UTC. Bloquea automáticamente issues y pull requests cerrados que no han recibido actividad en 14 días, evitando que discussions antiguas continúen sin supervisión. Solo cierra temas humanos; humanos deben intervenir para cerrar issues abiertos.

## Herramientas

- **uv**: gestor de paquetes y ejecutor de tareas Python, con soporte para lockfiles y caché
- **tox**: orquestador de pruebas que ejecuta la suite en entornos aislados
- **mypy**: verificador de tipos estáticos
- **ruff**: linter y formateador de código
- **pre-commit**: framework para hooks que se ejecutan antes de commits

## Decisiones

Los workflows ignoran cambios en documentación (`docs/**`, `README.md`) en jobs de pruebas. Esto evita ejecutar la suite completa cuando solo se actualizan docs, acelerando el feedback en cambios de documentación.
