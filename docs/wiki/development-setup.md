# Configuración del Entorno de Desarrollo

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Esta página describe cómo configurar tu entorno de desarrollo local para trabajar con itsdangerous, incluyendo contenedores, herramientas de linting y pruebas.

## Entorno con Dev Container

El proyecto incluye configuración para usar Dev Containers de VS Code, que proporciona un entorno completamente aislado y reproducible.

### Archivo de configuración

[`.devcontainer/devcontainer.json`](../../.devcontainer/devcontainer.json)

La configuración utiliza una imagen base de Python 3 de Microsoft y aplica las siguientes personalizaciones para VS Code:

- Ruta del intérprete: `.venv` en la carpeta del workspace
- Activación automática del entorno virtual en la terminal
- Flags de ejecución de Python en modo desarrollo (`-X dev`)

### Script de inicialización

[`.devcontainer/on-create-command.sh`](../../.devcontainer/on-create-command.sh)

El script que se ejecuta al crear el contenedor realiza los siguientes pasos:

1. Crea un entorno virtual con `.venv` y actualiza sus dependencias
2. Activa el entorno virtual
3. Instala las dependencias de desarrollo desde `requirements/dev.txt`
4. Instala el paquete itsdangerous en modo editable (desarrollo)
5. Configura pre-commit hooks automáticos

## Herramientas de Linting y Formateo

El proyecto usa **pre-commit** para validar el código antes de cada commit.

### Configuración de pre-commit

[`.pre-commit-config.yaml`](../../.pre-commit-config.yaml)

Las herramientas configuradas son:

- **ruff**: Linter rápido de Python que verifica código y aplica formato automático
  - `ruff`: valida problemas de estilo y lógica
  - `ruff-format`: formatea el código automáticamente
- **pre-commit-hooks**: conjunto de validaciones generales
  - `check-merge-conflict`: detecta marcadores de conflicto no resueltos
  - `debug-statements`: encuentra declaraciones de debug dejadas en el código
  - `fix-byte-order-marker`: elimina marcadores de orden de bytes innecesarios
  - `trailing-whitespace`: elimina espacios en blanco al final de líneas
  - `end-of-file-fixer`: asegura saltos de línea finales

Los hooks se actualizan automáticamente cada mes.

## Configuración del Editor

[`.editorconfig`](../../.editorconfig)

El proyecto incluye configuración estándar `.editorconfig` que establece:

- Indentación: 4 espacios para archivos Python, 2 para web
- Salto de línea: LF (Unix)
- Codificación: UTF-8
- Línea máxima: 88 caracteres (compatible con ruff)
- Espacios en blanco finales: eliminados
- Salto final de archivo: requerido

Esta configuración es respaldada por editores como VS Code, PyCharm y Sublime Text.

## Testing con Tox

[`tox.ini`](../../tox.ini)

El proyecto usa **tox** para ejecución reproducible de pruebas y validación en múltiples entornos.

### Entornos disponibles

**Pruebas:**
- `py3{12,11,10,9,8}`: Python 3.8 a 3.12
- `pypy310`: PyPy 3.10

Ejecuta: `tox` (todos los entornos) o `tox -e py311` (uno específico)

**Validación de código:**
- `style`: ejecuta pre-commit (ruff, formateo, checks)
- `typing`: ejecuta mypy y pyright para verificación de tipos estática

**Documentación:**
- `docs`: genera la documentación con Sphinx

**Mantenimiento:**
- `update-requirements`: actualiza archivos compilados de dependencias con pip-tools

### Características de tox

- Compila dependencias de requisitos usando `pip-compile`
- Salta intérpretes faltantes automáticamente (`skip_missing_interpreters`)
- Usa constraints congelados para reproducibilidad
- Construye ruedas para instalar el paquete en cada entorno

## Primeros pasos

1. **Con Dev Container**: abre el proyecto en VS Code con la extensión Dev Containers instalada; se pedirá automáticamente reabrir en el contenedor
2. **Sin Dev Container**: ejecuta manualmente los pasos de `on-create-command.sh` o usa `tox -e style` para validar

Consulta [Configuración del Proyecto](project-configuration.md) para detalles sobre estructura de dependencias y [Flujos de Integración Continua](ci-cd-pipelines.md) para cómo se valida automáticamente el código.
