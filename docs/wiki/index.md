# Visión general

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

ItsDangerous es una librería de Python que proporciona herramientas para transmitir datos de forma segura a entornos no confiables y recuperarlos sin alteraciones. [`README.md:7-9`](../../README.md#L7-L9) Los datos se firman criptográficamente para garantizar que no hayan sido modificados. [`README.md:11-13`](../../README.md#L11-L13)

## ¿Qué hace ItsDangerous?

La librería resuelve el problema de intercambiar datos seguros entre aplicaciones, especialmente en contextos web donde los tokens o datos deben viajar por canales potencialmente inseguros. Por ejemplo, permite generar un token con información de un usuario que puede ser enviado al cliente y posteriormente verificado sin riesgo de manipulación.

[`README.md:18-31`](../../README.md#L18-L31) Un ejemplo típico es serializar datos de un usuario, transmitirlos en una cookie o parámetro URL, y verificar su integridad al recibirlos:

```python
from itsdangerous import URLSafeSerializer
auth_s = URLSafeSerializer("secret key", "auth")
token = auth_s.dumps({"id": 5, "name": "itsdangerous"})
data = auth_s.loads(token)
```

## Componentes principales

ItsDangerous está construida sobre varios componentes que trabajan en conjunto:

```mermaid
graph TB
    A["Signer<br/>(Firma HMAC)"]
    B["URL Safe Encoder<br/>(Codificación segura)"]
    C["Serializer<br/>(Serialización)"]
    D["Timestamp Verifier<br/>(Verificación temporal)"]
    
    C --> A
    C --> B
    D --> A
    
    style A fill:#e1f5ff
    style B fill:#e1f5ff
    style C fill:#fff3e0
    style D fill:#f3e5f5
```

- **[Signer (Firma de datos)](signer.md)**: Sistema central que genera y verifica firmas HMAC. Garantiza la autenticidad de los datos usando una clave secreta.

- **[URL Safe Encoder (Codificación segura para URL)](url-safe-encoding.md)**: Codifica los datos en un formato seguro para transmitir en URLs y bases de datos, permitiendo caracteres especiales sin problemas.

- **[Serializer (Serialización y compresión)](serializer.md)**: Serializa objetos Python a formatos seguros (como JSON o pickle) e integra automáticamente la firma. Comprime los datos cuando es necesario.

- **[Timestamp Verifier (Verificación con marca de tiempo)](timestamp-verification.md)**: Extensión que añade marcas de tiempo a los tokens para controlar su validez temporal, permitiendo rechazar tokens antiguos.

## Cómo ejecutar el código

[`pyproject.toml:25-54`](../../pyproject.toml#L25-L54) Para configurar el entorno de desarrollo:

```bash
git clone https://github.com/pallets/itsdangerous
cd itsdangerous
python3 -m venv env
. env/bin/activate  # En Windows: env\Scripts\activate
uv pip install -e .
pre-commit install
```

[`pyproject.toml:138-158`](../../pyproject.toml#L138-L158) Para ejecutar las pruebas:

```bash
pytest              # Pruebas del entorno actual
tox                 # Suite completa de pruebas
```

## Guía de referencia

- **[API Reference (Referencia de API y conceptos)](api-reference.md)** — Conceptos fundamentales, excepciones y configuración general de la librería.

- **[Development Setup (Configuración y desarrollo)](development-setup.md)** — Herramientas de desarrollo, dependencias y configuración del entorno para contribuidores.

- **[CI and Deployment (Integración continua y publicación)](ci-and-deployment.md)** — Flujos de trabajo automatizados para pruebas, verificación de seguridad y publicación de versiones.

- **[Documentation (Sistema de documentación)](documentation.md)** — Cómo construir la documentación con Sphinx y ReadTheDocs.

- **[Changelog (Historial de cambios)](changelog.md)** — Registro de versiones y cambios significativos del proyecto.

## Decisiones

**Usar `uv` como gestor de dependencias** (11b0f7e)

Se migró de `pip` y `requirements/` a `uv` con dependency groups definidos en `pyproject.toml`. Esto simplifica la configuración del entorno de desarrollo y permite gestionar grupos de dependencias específicas (dev, tests, typing, docs, etc.) de forma declarativa.

**Usar guía de contribución global** (11b0f7e)

Se eliminó el archivo `CONTRIBUTING.rst` local para sincronizar con la documentación de contribución centralizada de Pallets en `https://palletsprojects.com/contributing/`, reduciendo la duplicación de documentación.
