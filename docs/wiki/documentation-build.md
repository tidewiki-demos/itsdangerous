# Generación de Documentación

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

El proyecto utiliza Sphinx para generar documentación y se configura para compilación automática en Read the Docs.

## Configuración de Sphinx

La documentación se construye usando Sphinx con las siguientes herramientas y extensiones [`.readthedocs.yaml:1-13`](../../.readthedocs.yaml#L1-L13):

- **Builder**: dirhtml, que genera documentación en formato HTML con directorios
- **Python**: versión 3.12 en Ubuntu 22.04
- **Modo estricto**: `fail_on_warning: true` garantiza que ninguna advertencia pase desapercibida

Las extensiones Sphinx configuradas son [`docs/conf.py:14-20`](../../docs/conf.py#L14-L20):

- `sphinx.ext.autodoc`: documentación automática desde docstrings de código
- `sphinx.ext.extlinks`: enlaces abreviados a recursos externos (issues, PRs)
- `sphinx.ext.intersphinx`: referencias cruzadas a documentación de Python
- `sphinxcontrib.log_cabinet`: gestión de registros de cambios
- `pallets_sphinx_themes`: tema personalizado para proyectos Pallets

## Dependencias

Las dependencias de documentación se especifican en `requirements/docs.txt`, que se instala automáticamente durante la compilación [`.readthedocs.yaml:8`](../../.readthedocs.yaml#L8).

## Construcción Local

El proyecto proporciona scripts para compilar documentación localmente:

- **En sistemas Unix/Linux/macOS**: [`docs/Makefile`](../../docs/Makefile) - usar `make` en el directorio `docs/`
- **En Windows**: [`docs/make.bat`](../../docs/make.bat) - ejecutar `make.bat`

Ambos scripts invocan `sphinx-build` con la ruta `docs/` como directorio fuente y `docs/_build` como directorio de salida.

## Estructura de Documentación

Los archivos RST se organizan en `docs/`:

- [`docs/index.rst`](../../docs/index.rst) - página principal con tabla de contenidos
- [`docs/concepts.rst`](../../docs/concepts.rst) - guías conceptuales fundamentales
- [`docs/signer.rst`](../../docs/signer.rst) - interfaz de firma criptográfica
- [`docs/serializer.rst`](../../docs/serializer.rst) - interfaz de serialización
- [`docs/timed.rst`](../../docs/timed.rst) - firma con marcas temporales
- [`docs/url_safe.rst`](../../docs/url_safe.rst) - serialización segura para URL
- [`docs/encoding.rst`](../../docs/encoding.rst) - utilidades de codificación
- [`docs/exceptions.rst`](../../docs/exceptions.rst) - referencia de excepciones
- [`docs/changes.rst`](../../docs/changes.rst) - incluye el archivo CHANGES.rst del proyecto
- [`docs/license.rst`](../../docs/license.rst) - incluye la licencia BSD-3-Clause

## Tema y Presentación

La documentación usa el tema "flask" de Pallets [`docs/conf.py:35-55`](../../docs/conf.py#L35-L55) con:

- Logos personalizados (`itsdangerous-logo-sidebar.png`)
- Enlaces de proyecto en la barra lateral (Donate, PyPI, GitHub, Discord)
- Generación automática de versión desde los metadatos del paquete
- Búsqueda y anuncios éticos habilitados

## Integración Continua

[`.readthedocs.yaml`](../../.readthedocs.yaml) muestra que la compilación:

1. Instala dependencias desde `requirements/docs.txt`
2. Instala el paquete localmente con pip
3. Ejecuta sphinx-build con el builder dirhtml
4. Falla si hay advertencias
