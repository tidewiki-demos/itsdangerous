# Guía de Contribución

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Esta guía explica cómo contribuir al proyecto ItsDangerous, incluyendo los estándares de código, el proceso de pull requests y la configuración del entorno de desarrollo.

## Preguntas y soporte

No uses el rastreador de problemas (issue tracker) para hacer preguntas sobre cómo usar ItsDangerous o para resolver problemas en tu propio código. El rastreador está destinado únicamente a reportar bugs y solicitar características en ItsDangerous mismo.

Para obtener ayuda, utiliza uno de los siguientes recursos:

- El canal `#get-help` en el chat de Discord de Pallets: https://discord.gg/pallets
- La lista de correo flask@python.org para discusiones a largo plazo o problemas más grandes
- Stack Overflow: busca primero con Google usando `site:stackoverflow.com itsdangerous {término de búsqueda}`

## Reportar problemas

Cuando reportes un issue, incluye la siguiente información:

- Describe qué esperabas que sucediera
- Si es posible, incluye un ejemplo mínimo reproducible que ayude a identificar el problema
- Describe qué sucedió realmente, incluyendo el traceback completo si hubo una excepción
- Lista tu versión de Python e ItsDangerous
- Verifica si el problema ya está corregido en las versiones más recientes o en el código más reciente del repositorio

Las plantillas de issue [`.github/ISSUE_TEMPLATE/bug-report.md:1-27`](../../.github/ISSUE_TEMPLATE/bug-report.md#L1-L27) y [`.github/ISSUE_TEMPLATE/feature-request.md:1-15`](../../.github/ISSUE_TEMPLATE/feature-request.md#L1-L15) guían este proceso automáticamente.

## Envío de cambios (patches)

### Antes de comenzar

Si no existe un issue abierto para lo que deseas enviar, es preferible abrir uno primero para discutirlo antes de trabajar en un pull request. Puedes trabajar en cualquier issue que no tenga un PR abierto vinculado o un mantenedor asignado.

### Requisitos para tu parche

[`CONTRIBUTING.rst:52-63`](../../CONTRIBUTING.rst#L52-L63) establece que tu parche debe incluir:

- Código formateado con Black
- Tests si tu cambio añade o modifica código (los tests deben fallar sin tu cambio)
- Actualización de las páginas de documentación relevantes y docstrings (con envoltura a 72 caracteres)
- Una entrada en `CHANGES.rst` siguiendo el mismo estilo que otras entradas
- Entradas `.. versionchanged::` en los docstrings relevantes

Se recomienda instalar pre-commit para que estas herramientas se ejecuten automáticamente.

## Configuración inicial

### Setup del repositorio

[`CONTRIBUTING.rst:72-129`](../../CONTRIBUTING.rst#L72-L129) describe los pasos iniciales:

1. Descarga e instala la última versión de git
2. Configura git con tu nombre y email:
   ```
   $ git config --global user.name 'tu nombre'
   $ git config --global user.email 'tu email'
   ```
3. Crea una cuenta en GitHub
4. Fork ItsDangerous haciendo clic en el botón Fork
5. Clona el repositorio principal localmente:
   ```
   $ git clone https://github.com/pallets/itsdangerous
   $ cd itsdangerous
   ```
6. Añade tu fork como remoto (reemplaza `{username}` con tu usuario):
   ```
   $ git remote add fork https://github.com/{username}/itsdangerous
   ```

### Entorno de desarrollo

[`CONTRIBUTING.rst:98-122`](../../CONTRIBUTING.rst#L98-L122) especifica cómo configurar el entorno:

1. Crea un virtualenv:
   ```
   $ python3 -m venv env
   $ . env/bin/activate
   ```
   (En Windows: `env\Scripts\activate`)

2. Instala las dependencias de desarrollo:
   ```
   $ pip install -r requirements/dev.txt && pip install -e .
   ```

3. Instala los pre-commit hooks:
   ```
   $ pre-commit install
   ```

## Flujo de trabajo para desarrolladores

### Crear una rama

[`CONTRIBUTING.rst:132-150`](../../CONTRIBUTING.rst#L132-L150) describe cómo crear tu rama de trabajo:

- Para correcciones de bugs o documentación, crea una rama basada en la rama ".x" más reciente:
  ```
  $ git fetch origin
  $ git checkout -b tu-nombre-rama origin/2.0.x
  ```

- Para nuevas características o cambios, crea una rama basada en la rama "main":
  ```
  $ git fetch origin
  $ git checkout -b tu-nombre-rama origin/main
  ```

### Hacer commits y pruebas

[`CONTRIBUTING.rst:152-165`](../../CONTRIBUTING.rst#L152-L165) indica que debes:

1. Hacer cambios usando tu editor favorito
2. Confirmar tus cambios conforme avanzas
3. Incluir tests que cubran cualquier cambio de código (asegúrate de que los tests fallen sin tu parche)
4. Ejecutar los tests localmente (ver sección siguiente)
5. Hacer push de tus commits a tu fork en GitHub:
   ```
   $ git push --set-upstream fork tu-nombre-rama
   ```
6. Crear un pull request vinculándolo al issue con `fixes #123`

## Ejecutar pruebas

### Tests básicos

[`CONTRIBUTING.rst:171-175`](../../CONTRIBUTING.rst#L171-L175) explica cómo ejecutar la suite de tests:

```
$ pytest
```

Esto ejecuta los tests para el entorno actual, lo que generalmente es suficiente. CI ejecutará la suite completa cuando envíes tu pull request.

### Suite completa con tox

Si prefieres no esperar a que CI ejecute todo, puedes usar tox:

```
$ tox
```

### Cobertura de tests

[`CONTRIBUTING.rst:187-202`](../../CONTRIBUTING.rst#L187-L202) describe cómo generar un reporte de cobertura:

```
$ pip install coverage
$ coverage run -m pytest
$ coverage html
```

Abre `htmlcov/index.html` en tu navegador para explorar el reporte. Esto puede indicarte dónde empezar a contribuir enfocándote en líneas sin cobertura de tests.

## Construir la documentación

[`CONTRIBUTING.rst:205-216`](../../CONTRIBUTING.rst#L205-L216) explica cómo compilar la documentación:

```
$ cd docs
$ make html
```

Abre `_build/html/index.html` en tu navegador para ver la documentación compilada.

## Plantilla de Pull Request

Cuando crees un PR, asegúrate de seguir los pasos indicados en [`.github/pull_request_template.md:1-24`](../../.github/pull_request_template.md#L1-L24):

- Abre un ticket describiendo el issue o característica que tu PR abordará (no es necesario para cambios simples como correcciones de typos)
- Enlaza al issue relevante con `fixes #<issue number>`
- Verifica que cada paso en CONTRIBUTING.rst esté completo:
  - Tests que demuestren el comportamiento correcto (deben fallar sin tu cambio)
  - Documentación añadida o actualizada
  - Entrada en CHANGES.rst
  - Entradas `.. versionchanged::` en docstrings relevantes
