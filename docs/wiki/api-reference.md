# Referencia de API y conceptos

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Documentación de los conceptos fundamentales de la librería ItsDangerous, incluyendo las dos niveles principales de operación (Signer y Serializer), la gestión de claves secretas, y las excepciones que se pueden generar durante la operación.

## Conceptos fundamentales

### Signer vs Serializer

ItsDangerous proporciona dos niveles de manejo de datos [`docs/concepts.rst:5-11`](../../docs/concepts.rst#L5-L11):

- **Signer**: el sistema básico que firma un valor `bytes` basándose en los parámetros de firma proporcionados. Véase [Firma de datos](signer.md).
- **Serializer**: envuelve un signer para permitir serializar y firmar datos además de `bytes`. Véase [Serialización y compresión](serializer.md).

En la mayoría de los casos, querrás usar un serializer en lugar de un signer, ya que permite configurar los parámetros de firma y proporcionar signers alternativos para actualizar tokens antiguos a nuevos parámetros [`docs/concepts.rst:13-15`](../../docs/concepts.rst#L13-L15).

### La clave secreta

Las firmas se aseguran mediante la `secret_key`. Típicamente, se usa una única clave secreta con todos los signers, y el salt se utiliza para distinguir diferentes contextos [`docs/concepts.rst:18-29`](../../docs/concepts.rst#L18-L29).

La clave secreta:
- Debe ser una cadena larga y aleatoria de bytes
- Debe mantenerse en secreto y no guardarse en el código fuente ni confirmarse en control de versiones
- Si un atacante la descubre, puede cambiar y refirmar datos para que parezcan válidos
- Si sospechas que fue comprometida, debes cambiarla para invalidar los tokens existentes

Se recomienda leer la clave secreta desde una variable de entorno [`docs/concepts.rst:31-35`](../../docs/concepts.rst#L31-L35):

```python
import os
from itsdangerous.serializer import Serializer
SECRET_KEY = os.environ.get("SECRET_KEY")
s = Serializer(SECRET_KEY)
```

Una forma de generar una clave es usar `os.urandom` [`docs/concepts.rst:49-53`](../../docs/concepts.rst#L49-L53):

```bash
$ python3 -c 'import os; print(os.urandom(16).hex())'
```

### El salt

El salt se combina con la clave secreta para derivar una clave única que distingue diferentes contextos [`docs/concepts.rst:56-62`](../../docs/concepts.rst#L56-L62). A diferencia de la clave secreta:

- No tiene que ser aleatorio
- Puede guardarse en el código
- Solo tiene que ser único entre contextos, no privado

**Caso de uso**: si deseas enviar enlaces de activación de correo electrónico y enlaces de actualización, y solo firmas el ID de usuario sin usar salts diferentes, un usuario podría reutilizar el token de un contexto en otro [`docs/concepts.rst:64-69`](../../docs/concepts.rst#L64-L69). Con salts diferentes, las firmas serán distintas:

```python
from itsdangerous.url_safe import URLSafeSerializer

s1 = URLSafeSerializer("secret-key", salt="activate")
s1.dumps(42)
# 'NDI.MHQqszw6Wc81wOBQszCrEE_RlzY'

s2 = URLSafeSerializer("secret-key", salt="upgrade")
s2.dumps(42)
# 'NDI.c0MpsD6gzpilOAeUPra3NShPXsE'

# s2 no puede cargar datos firmados por s1
s2.loads(s1.dumps(42))  # BadSignature: Signature does not match

# Solo el serializer con el mismo salt puede cargar los datos
s2.loads(s2.dumps(42))  # 42
```

### Rotación de claves

La rotación de claves proporciona una capa adicional de mitigación contra un atacante que descubre una clave secreta [`docs/concepts.rst:101-110`](../../docs/concepts.rst#L101-L110). En lugar de pasar una única clave, puedes pasar una lista de claves (de más antigua a más nueva):

- Al firmar, se usa la última clave (más nueva)
- Al validar, se prueba cada clave desde la más nueva hasta la más antigua antes de generar un error de validación

[`docs/concepts.rst:121-134`](../../docs/concepts.rst#L121-L134):

```python
SECRET_KEYS = ["2b9cd98e", "169d7886", "b6af09f5"]

# firmar datos con la clave más reciente
s = Serializer(SECRET_KEYS)
t = s.dumps({"id": 42})

# rotar una clave nueva y eliminar la más antigua
SECRET_KEYS.append("cf9b3588")
del SECRET_KEYS[0]

s = Serializer(SECRET_KEYS)
s.loads(t)  # válido aunque fue firmado con una clave anterior
```

### Seguridad del método de digest

Un signer se configura con un `digest_method`, una función hash usada como paso intermedio al generar la firma HMAC [`docs/concepts.rst:140-144`](../../docs/concepts.rst#L140-L144). El método por defecto es `hashlib.sha1`.

Los usuarios a veces se preocupan por esto debido a colisiones de hash conocidas en SHA-1. Sin embargo, cuando se usa como paso iterado intermedio en HMAC, SHA-1 no es inseguro. De hecho, incluso MD5 sigue siendo seguro en HMAC: la seguridad del hash por sí solo no se aplica cuando se usa en HMAC [`docs/concepts.rst:146-148`](../../docs/concepts.rst#L146-L148).

Si tu proyecto considera SHA-1 un riesgo, puedes configurar el signer con un método de digest diferente como `hashlib.sha512`. Ten en cuenta que SHA-512 produce un hash más largo, lo que hace que los tokens ocupen más espacio (relevante en cookies y URLs) [`docs/concepts.rst:150-154`](../../docs/concepts.rst#L150-L154).

## Excepciones

La librería define las siguientes excepciones [`src/itsdangerous/__init__.py:8-13`](../../src/itsdangerous/__init__.py#L8-L13):

| Excepción | Descripción |
|-----------|-------------|
| `BadData` | Excepción base para datos inválidos |
| `BadSignature` | La firma no es válida o no coincide |
| `BadTimeSignature` | La firma relacionada con tiempo es inválida |
| `SignatureExpired` | La firma ha expirado (cuando se usa con marca de tiempo) |
| `BadHeader` | El encabezado de los datos es inválido |
| `BadPayload` | La carga útil de los datos es inválida |

Consulta [Verificación con marca de tiempo](timestamp-verification.md) para casos de uso con expiración de firmas.

## API principal

[`src/itsdangerous/__init__.py`](../../src/itsdangerous/__init__.py) expone los siguientes componentes:

- **Signers**: `Signer`, `HMACAlgorithm`, `NoneAlgorithm`
- **Serializers**: `Serializer`, `URLSafeSerializer`, `TimedSerializer`, `URLSafeTimedSerializer`
- **Marca de tiempo**: `TimestampSigner`
- **Utilidades de codificación**: `base64_decode`, `base64_encode`, `want_bytes`
- **Excepciones**: `BadData`, `BadSignature`, `BadTimeSignature`, `SignatureExpired`, `BadHeader`, `BadPayload`

Consulta las páginas específicas para detalles:
- [Firma de datos](signer.md)
- [Serialización y compresión](serializer.md)
- [Verificación con marca de tiempo](timestamp-verification.md)
- [Codificación segura para URL](url-safe-encoding.md)
