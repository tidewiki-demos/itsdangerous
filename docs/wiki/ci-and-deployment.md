# Integración continua y publicación

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Este repositorio utiliza flujos de trabajo automatizados en GitHub Actions para ejecutar pruebas, verificar dependencias y publicar versiones. Los flujos están definidos en `.github/workflows/` y se disparan con diferentes eventos (push, pull requests, tags y cronograma).

## Pruebas automatizadas

[`.github/workflows/tests.yaml:1-7`](../../.github/workflows/tests.yaml#L1-L7)

El flujo `Tests` se ejecuta en cada push a `main` y rama `stable`, así como en pull requests. Ignora cambios que afecten solo documentación y el archivo README.

Las pruebas se ejecutan en una matriz de configuraciones:
- **Versiones de Python**: 3.13, 3.12, 3.11, 3.10 y PyPy 3.11
- **Sistemas operativos**: Ubuntu (por defecto), Windows y macOS para Python 3.13

[`.github/workflows/tests.yaml:32`](../../.github/workflows/tests.yaml#L32)

Cada combinación ejecuta la suite de pruebas mediante `tox` con `uv run --locked`.

[`.github/workflows/tests.yaml:33-49`](../../.github/workflows/tests.yaml#L33-L49)

El flujo también incluye un trabajo `typing` que valida la integridad de tipos usando `mypy`, con caché persistente para optimizar las ejecuciones posteriores.

## Validación de pre-commit

[`.github/workflows/pre-commit.yaml`](../../.github/workflows/pre-commit.yaml)

Un flujo ejecuta en cada pull request y push a `main` y `stable` para validar el código con `pre-commit`. Utiliza `uv` para ejecutar las herramientas de verificación definidas en `.pre-commit-config.yaml`.

## Bloqueo de issues inactivos

[`.github/workflows/lock.yaml`](../../.github/workflows/lock.yaml)

Un flujo programado ejecuta diariamente a las 00:00 UTC para bloquear issues, pull requests y discusiones cerrados sin actividad durante 14 días. Esto limita discusiones en hilos antiguos, priorizando que nuevas preguntas se resuelvan en issues actuales.

## Publicación de versiones

[`.github/workflows/publish.yaml:1-6`](../../.github/workflows/publish.yaml#L1-L6)

El flujo `Publish` se dispara cuando se crea un tag en el repositorio. Implementa un proceso de varios pasos:

### Construcción

[`.github/workflows/publish.yaml:6-21`](../../.github/workflows/publish.yaml#L6-L21)

El trabajo `build` compila los artefactos (sdist y wheels) usando `uv build`. Establece `SOURCE_DATE_EPOCH` desde la fecha del commit para construir de manera reproducible.

### Creación de release

[`.github/workflows/publish.yaml:22-34`](../../.github/workflows/publish.yaml#L22-L34)

El trabajo `create-release` descarga los artefactos y crea un release en GitHub como borrador (draft), incluyendo los archivos compilados. Esto permite revisar los artefactos antes de la publicación definitiva.

### Publicación a PyPI

[`.github/workflows/publish.yaml:35-47`](../../.github/workflows/publish.yaml#L35-L47)

El trabajo `publish-pypi` requiere aprobación manual mediante un environment denominado `publish` antes de subir los paquetes a PyPI. Utiliza OpenID Connect para autenticación sin necesidad de tokens estáticos.

## Decisiones

**Construcción reproducible**: Se captura `SOURCE_DATE_EPOCH` del commit para que las compilaciones posteriores del mismo tag produzcan artefactos idénticos. [`.github/workflows/publish.yaml:17`](../../.github/workflows/publish.yaml#L17)

**Aprobación manual antes de publicación**: La publicación a PyPI requiere revisión humana del contenido del release antes de liberarse públicamente, reduciendo riesgos de publicación accidental de versiones problemáticas. [`.github/workflows/publish.yaml:37-39`](../../.github/workflows/publish.yaml#L37-L39)

**Eliminación de provenance SLSA**: Se removió la generación de prueba SLSA 3 ya que PyPI y su sistema de publicación confiable (trusted publishing) incluyen soporte de atestación integrado. Esto simplifica el flujo de publicación sin comprometer la seguridad. Commit 584138c73af1.

**Migración a uv**: Los flujos de trabajo ahora utilizan `uv` como gestor de dependencias y ejecutor de tareas en lugar de pip y ejecución directa, mejorando la velocidad y la reproducibilidad. Commit 91952b90cd5c.

**Eliminación de versiones Python antiguas**: Se dejaron de soportar Python 3.9, 3.8 y PyPy 3.10 para enfocarse en versiones modernas. La matriz de pruebas ahora cubre Python 3.10-3.13 y PyPy 3.11. Commit 7edfa12ed562.
