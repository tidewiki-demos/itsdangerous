# Integración continua y publicación

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Este repositorio utiliza flujos de trabajo automatizados en GitHub Actions para ejecutar pruebas, verificar dependencias y publicar versiones. Los flujos están definidos en `.github/workflows/` y se disparan con diferentes eventos (push, pull requests, tags y cronograma).

## Pruebas automatizadas

[`.github/workflows/tests.yaml:1-41`](../../.github/workflows/tests.yaml#L1-L41)

El flujo `Tests` se ejecuta en cada push a `main` y ramas de la serie `*.x`, así como en pull requests. Ignora cambios que afecten solo documentación y archivos Markdown.

Las pruebas se ejecutan en una matriz de configuraciones:
- **Versiones de Python**: 3.13, 3.12, 3.11, 3.10, 3.9, 3.8 y PyPy 3.10
- **Sistemas operativos**: Ubuntu (por defecto), Windows y macOS para Python 3.12

Cada combinación ejecuta la suite de pruebas mediante `tox`. [`.github/workflows/tests.yaml:40-41`](../../.github/workflows/tests.yaml#L40-L41)

[`.github/workflows/tests.yaml:42-57`](../../.github/workflows/tests.yaml#L42-L57)

El flujo también incluye un trabajo `typing` que valida la integridad de tipos usando `mypy`, con caché persistente para optimizar las ejecuciones posteriores.

## Bloqueo de issues inactivos

[`.github/workflows/lock.yaml`](../../.github/workflows/lock.yaml)

Un flujo programado ejecuta diariamente a las 00:00 UTC para bloquear issues y pull requests cerrados sin actividad durante 14 días. Esto limita discusiones en hilos antiguos, priorizando que nuevas preguntas se resuelvan en issues actuales.

## Publicación de versiones

[`.github/workflows/publish.yaml:1-6`](../../.github/workflows/publish.yaml#L1-L6)

El flujo `Publish` se dispara cuando se crea un tag en el repositorio. Implementa un proceso de varios pasos:

### Construcción y generación de hashes

[`.github/workflows/publish.yaml:7-28`](../../.github/workflows/publish.yaml#L7-L28)

El trabajo `build` compila los artefactos (sdist y wheels) usando `python -m build`. Establece `SOURCE_DATE_EPOCH` desde la fecha del commit para construir de manera reproducible. Genera hashes SHA-256 en base64 para los artefactos, que se utilizan para generar prueba de provenance.

### Prueba de provenance SLSA

[`.github/workflows/publish.yaml:29-38`](../../.github/workflows/publish.yaml#L29-L38)

El trabajo `provenance` utiliza el generador SLSA de GitHub para crear una declaración SLSA 3 que atestigua la construcción de los artefactos. Esto mejora la seguridad al demostrar que los binarios provienen de este repositorio específico.

### Creación de release

[`.github/workflows/publish.yaml:39-54`](../../.github/workflows/publish.yaml#L39-L54)

El trabajo `create-release` descarga los artefactos y crea un release en GitHub como borrador (draft), incluyendo los archivos `.intoto.jsonl` de provenance. Esto permite revisar los artefactos antes de la publicación definitiva.

### Publicación a PyPI

[`.github/workflows/publish.yaml:55-73`](../../.github/workflows/publish.yaml#L55-L73)

El trabajo `publish-pypi` requiere aprobación manual mediante un environment denominado `publish` antes de subir los paquetes. Publica primero en TestPyPI para validación, luego en PyPI oficial. Ambas cargas utilizan OpenID Connect para autenticación sin necesidad de tokens estáticos.

## Decisiones

**Construcción reproducible**: Se captura `SOURCE_DATE_EPOCH` del commit para que las compilaciones posteriores del mismo tag produzcan artefactos idénticos. [`.github/workflows/publish.yaml:19-20`](../../.github/workflows/publish.yaml#L19-L20)

**Aprobación manual antes de publicación**: La publicación a PyPI requiere revisión humana del contenido del release antes de liberarse públicamente, reduciendo riesgos de publicación accidental de versiones problemáticas. [`.github/workflows/publish.yaml:57-60`](../../.github/workflows/publish.yaml#L57-L60)

**Provenance SLSA**: Se genera prueba criptográfica de compilación para permitir a los consumidores verificar que los paquetes fueron construidos por este repositorio, incrementando la confiabilidad de la cadena de suministro. [`.github/workflows/publish.yaml:29-38`](../../.github/workflows/publish.yaml#L29-L38)
