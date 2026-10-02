# Sistema de documentación

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

La documentación del proyecto se construye con Sphinx y se publica automáticamente en ReadTheDocs. La configuración combina herramientas estándar de la comunidad Python con temas y extensiones especializadas.

## Configuración de ReadTheDocs

[`.readthedocs.yaml`](../../.readthedocs.yaml)

El archivo `.readthedocs.yaml` define cómo se construye la documentación en el servicio ReadTheDocs:

- **Versión de compilación**: Usa Ubuntu 24.04 con Python 3.13
- **Gestor de paquetes**: Utiliza `uv` como gestor de dependencias [`.readthedocs.yaml:6-9`](../../.readthedocs.yaml#L6-L9)
- **Constructor Sphinx**: Ejecuta `sphinx-build` con la configuración de `docs/conf.py`, indicador `-W` para tratar advertencias como errores, y builder `dirhtml` para generar documentación con estructura de directorios [`.readthedocs.yaml:10`](../../.readthedocs.yaml#L10)

Esta configuración asegura que la documentación se mantenga sin advertencias y con un nivel consistente de calidad.

## Construcción local

[`docs/Makefile`](../../docs/Makefile)[`docs/make.bat`](../../docs/make.bat)

Para construir la documentación localmente:

**En sistemas Unix/Linux/macOS:**
```bash
cd docs
make html
```

**En Windows:**
```cmd
cd docs
make.bat html
```

El `Makefile` y `make.bat` delegan a `sphinx-build`. Por defecto, la documentación se genera en el directorio `docs/_build`.

## Configuración de Sphinx

[`docs/conf.py`](../../docs/conf.py)

El archivo `conf.py` define el comportamiento del generador Sphinx:

**Metadatos del proyecto:**
- Nombre: "ItsDangerous"
- Copyright y autor: Pallets
- Versión obtenida dinámicamente desde el paquete instalado

**Extensiones habilitadas:**
- `sphinx.ext.autodoc`: Genera documentación automáticamente desde docstrings del código
- `sphinx.ext.extlinks`: Define atajos para enlaces externos (`:issue:`, `:pr:`)
- `sphinx.ext.intersphinx`: Vincula referencias cruzadas con la documentación de Python oficial
- `sphinxcontrib.log_cabinet`: Gestiona historial de cambios
- `pallets_sphinx_themes`: Proporciona tema personalizado

**Comportamiento de autodoc:**
- `autoclass_content = "both"`: Incluye docstrings de clase e `__init__`
- `autodoc_member_order = "bysource"`: Mantiene el orden de definición en el código
- `autodoc_typehints = "description"`: Coloca hints de tipo en la descripción
- `autodoc_preserve_defaults = True`: Conserva los valores por defecto en las firmas

**Tema y presentación:**
- Tema Flask de Pallets
- Logo y favicon personalizados en formato SVG [`docs/conf.py:52-53`](../../docs/conf.py#L52-L53)
- Barras laterales configuradas con búsqueda y enlaces del proyecto
- Enlaces contextuales a PyPI, repositorio, rastreador de issues y chat

## Flujo de publicación

Cuando se hace push a la rama principal, ReadTheDocs:
1. Clona el repositorio
2. Lee `.readthedocs.yaml`
3. Instala `uv` como gestor de paquetes
4. Ejecuta `sphinx-build` con la configuración de `docs/conf.py`
5. Publica el sitio HTML generado

Cualquier advertencia de Sphinx causa que la compilación falle, previniendo la publicación de documentación incompleta o problemática.

## Dependencias de documentación

Las dependencias necesarias para compilar la documentación se especifican mediante un grupo `docs` en la configuración de `uv`. ReadTheDocs instala estas dependencias usando `uv run --group docs` [`.readthedocs.yaml:10`](../../.readthedocs.yaml#L10).

## Decisiones

**Migración a `uv` como gestor de paquetes** (commit 91952b90cd5c): El proyecto cambió de pip a `uv` para gestionar dependencias. ReadTheDocs se configuró para instalar y usar `uv` a través de `asdf`, ejecutando comandos Sphinx con `uv run --group docs`.

**Logo en formato SVG** (commits 38774c9ae5f7 y f593a0584f2e): El favicon y el logo ahora utilizan archivos SVG (`itsdangerous-icon.svg` y `itsdangerous-logo.svg`) en lugar de PNG, mejorando la escalabilidad y calidad en diferentes tamaños.
