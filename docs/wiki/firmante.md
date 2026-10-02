# Firmante (Signer)

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Componente central que genera y verifica firmas criptográficas para garantizar la integridad de datos sin modificaciones. La clase `Signer` permite adjuntar una firma a un valor y posteriormente validar que el contenido no ha sido alterado.

## Concepto básico

El firmante opera según un patrón simple: toma un valor y una clave secreta, genera una firma criptográfica, y la adjunta al valor separada por un delimitador. Cualquier cambio en el valor hace que la firma sea inválida [`docs/signer.rst:6-22`](../../docs/signer.rst#L6-L22).

```python
from itsdangerous import Signer
s = Signer("secret-key")
signed = s.sign("my string")      # b'my string.wh6tMHxLgJqB6oY1uT73iMlyrOA'
original = s.unsign(signed)        # b'my string'
```

Si el valor se modifica, `unsign()` genera una excepción `BadSignature` [`docs/signer.rst:28-36`](../../docs/signer.rst#L28-L36).

## Clase Signer

La clase `Signer` es el punto de entrada principal [`src/itsdangerous/signer.py:76-112`](../../src/itsdangerous/signer.py#L76-L112). Sus parámetros clave son:

- **`secret_key`**: Clave secreta para firmar y verificar. Puede ser una cadena, bytes, o una lista de claves para rotación [`src/itsdangerous/signer.py:85-86`](../../src/itsdangerous/signer.py#L85-L86).
- **`salt`**: Valor adicional combinado con la clave secreta para distinguir firmas en diferentes contextos. Por defecto es `b"itsdangerous.Signer"` [[cite:src/itsdangerous/signer.py:87-88, 153-158]].
- **`sep`**: Separador entre el valor y la firma (por defecto `.`). Debe evitar caracteres base64 [[cite:src/itsdangerous/signer.py:89, 144-151]].
- **`key_derivation`**: Estrategia para derivar la clave de firma. Opciones: `concat`, `django-concat`, `hmac`, `none`. Por defecto `django-concat` [[cite:src/itsdangerous/signer.py:90-93, 127]].
- **`digest_method`**: Función hash para HMAC (por defecto SHA-1) [`src/itsdangerous/signer.py:94-97`](../../src/itsdangerous/signer.py#L94-L97).
- **`algorithm`**: Instancia de `SigningAlgorithm` personalizada [`src/itsdangerous/signer.py:98-100`](../../src/itsdangerous/signer.py#L98-L100).

## Algoritmos de firma

La clase base `SigningAlgorithm` define la interfaz que todo algoritmo debe implementar [`src/itsdangerous/signer.py:15-28`](../../src/itsdangerous/signer.py#L15-L28):

- `get_signature(key, value)`: Genera la firma para una clave y valor.
- `verify_signature(key, value, sig)`: Verifica que una firma sea válida usando comparación resistente a ataques de timing con `hmac.compare_digest`.

### HMACAlgorithm

Implementación por defecto que genera firmas usando HMAC [`src/itsdangerous/signer.py:48-64`](../../src/itsdangerous/signer.py#L48-L64). Usa SHA-1 por defecto pero es configurable con cualquier función de `hashlib`.

### NoneAlgorithm

Algoritmo que retorna una firma vacía sin realizar validación real [`src/itsdangerous/signer.py:31-37`](../../src/itsdangerous/signer.py#L31-L37). Útil para casos donde se requiere la interfaz pero no la seguridad.

## Derivación de claves

El método `derive_key()` combina la clave secreta con la sal usando diferentes estrategias [`src/itsdangerous/signer.py:182-213`](../../src/itsdangerous/signer.py#L182-L213):

- **`concat`**: `hash(salt + secret_key)`
- **`django-concat`**: `hash(salt + b"signer" + secret_key)` (por defecto)
- **`hmac`**: `HMAC(secret_key, salt)`
- **`none`**: Retorna la clave secreta sin modificación

## Métodos principales

[`src/itsdangerous/signer.py:215-256`](../../src/itsdangerous/signer.py#L215-L256)

- **`sign(value)`**: Retorna el valor con su firma adjunta separados por `sep`.
- **`unsign(signed_value)`**: Extrae el valor original verificando la firma. Lanza `BadSignature` si es inválida.
- **`get_signature(value)`**: Retorna solo la firma en formato base64.
- **`verify_signature(value, sig)`**: Valida una firma sin extraer el valor. Retorna booleano.
- **`validate(signed_value)`**: Valida el valor firmado sin lanzar excepciones. Retorna booleano.

## Rotación de claves

Se soporta rotación de claves pasando una lista al constructor. La clave más nueva (última en la lista) se usa para firmar, pero todas se aceptan para verificación [[cite:src/itsdangerous/signer.py:86, 138-143, 236-242]]:

```python
signer = Signer(["old-key", "new-key"])  # new-key para firmar
# Acepta firmas de ambas claves durante verificación
```

## Manejo de bytes y strings

La clase convierte automáticamente strings a UTF-8 usando la función `want_bytes()` [`docs/signer.rst:24-26`](../../docs/signer.rst#L24-L26). El resultado de `unsign()` siempre es `bytes`, sin forma de distinguir si el original era string o bytes.

## Relaciones con otros componentes

- Usa [Codificación y Excepciones](codificacion-y-excepciones.md) para `want_bytes()`, `base64_encode/decode()` y `BadSignature`.
- Forma la base de [Serializador](serializador.md), que añade serialización de tipos.
- Es extendida por [Firmante con Tiempo](firmante-con-tiempo.md) para registrar y validar antigüedad.
- Se integra con [Codificación URL Segura](url-segura.md) para casos de uso en URLs.

Véase [Conceptos Fundamentales](conceptos-fundamentales.md) para información sobre seguridad de claves secretas y sales.
