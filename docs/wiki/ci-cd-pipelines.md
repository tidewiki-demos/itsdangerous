# Flujos de Integración Continua

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Este repositorio configura tres flujos de trabajo principales en GitHub Actions que automatizan pruebas, publicación de releases y tareas de mantenimiento.

## Flujo de Pruebas

[`.github/workflows/tests.yaml:1-41`](../../.github/workflows/tests.yaml#L1-L41) El flujo "Tests" se ejecuta automáticamente en cada push a `main` y ramas `*.x`, así como en pull requests. Omite cambios que afecten solo documentación (archivos en `docs/`, `.md` y `.rst`).

Las pruebas se ejecutan en [`.github/workflows/tests.yaml:22-31`](../../.github/workflows/tests.yaml#L22-L31) una matriz de versiones de Python (3.8, 3.9, 3.10, 3.11, 3.12 y PyPy 3.10) y sistemas operativos (Ubuntu, Windows, macOS). Cada combinación instala dependencias desde `requirements/` y ejecuta tox con el entorno correspondiente mediante [`.github/workflows/tests.yaml:40-41`](../../.github/workflows/tests.yaml#L40-L41) `pip install tox` y `tox run`.

[`.github/workflows/tests.yaml:42-56`](../../.github/workflows/tests.yaml#L42-L56) En paralelo, un trabajo de análisis de tipos ejecuta mypy mediante tox con cacheo de resultados previos.

## Flujo de Publicación

[`.github/workflows/publish.yaml:1-5`](../../.github/workflows/publish.yaml#L1-L5) El flujo "Publish" se activa cuando se crea una etiqueta (tag) en el repositorio.

**Construcción del artefacto:** [`.github/workflows/publish.yaml:7-28`](../../.github/workflows/publish.yaml#L7-L28) Se compila el paquete usando Python 3.x, establece la fecha de compilación desde el commit mediante `SOURCE_DATE_EPOCH`, construye distribuciones con `python -m build` y genera hashes SHA256 en base64 para trazabilidad.

**Generación de provenance:** [`.github/workflows/publish.yaml:29-38`](../../.github/workflows/publish.yaml#L29-L38) Se utiliza SLSA Framework para generar un documento de provenance que certifica la autenticidad del build.

**Creación de release:** [`.github/workflows/publish.yaml:39-54`](../../.github/workflows/publish.yaml#L39-L54) Se descarga el artefacto y se crea un borrador de release en GitHub con los ficheros `.intoto.jsonl` (provenance) y distribuciones.

**Publicación en PyPI:** [`.github/workflows/publish.yaml:55-73`](../../.github/workflows/publish.yaml#L55-L73) Tras revisión manual en el environment `publish`, se sube el paquete primero a test.pypi.org y luego a PyPI de producción usando OIDC para autenticación.

## Flujo de Bloqueo de Issues

[`.github/workflows/lock.yaml:1-22`](../../.github/workflows/lock.yaml#L1-L22) El flujo "Lock inactive closed issues" se ejecuta diariamente. Bloquea automáticamente issues cerrados y pull requests que no han recibido actividad durante 14 días, evitando comentarios innecesarios en discusiones antiguas.
