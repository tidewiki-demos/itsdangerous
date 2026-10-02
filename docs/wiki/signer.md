# Firma de datos

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Sistema central de firma criptográfica que genera y verifica firmas HMAC para garantizar que los datos no hayan sido alterados. El módulo proporciona un mecanismo seguro de autenticación de mensajes basado en claves secretas.

## Conceptos básicos

La clase `Signer` [`src/itsdangerous/signer.py:76-112`](../../src/itsdangerous/signer.py#L76-L112) es el componente principal. Recibe una clave secreta y puede firmar datos agregando una firma criptográfica, o verificar que una firma es válida.

Cuando firmas datos, se genera una firma que se adjunta al dato original separada por un punto:

```python
from itsdangerous import Signer
s = Signer("secret-key")
s.sign("my string")
# b'my string.wh6tMHxLgJqB6oY1uT73iMlyrOA'
```

Para verificar, se usa `unsign()` que devuelve el dato original si la firma es válida o lanza `BadSignature` si fue alterado:

```python
s.unsign(b"my string.wh6tMHxLgJqB6oY1uT73iMlyrOA")
# b'my string'

s.unsign(b"different string.wh6tMHxLgJqB6oY1uT73iMlyrOA")
# BadSignature: Signature does not match
```

[`docs/signer.rst:6-37`](../../docs/signer.rst#L6-L37)

## Algoritmos de firma

El sistema soporta distintos algoritmos de firma mediante la clase base `SigningAlgorithm` [`src/itsdangerous/signer.py:15-28`](../../src/itsdangerous/signer.py#L15-L28):

- **HMACAlgorithm**: Genera firmas HMAC usando una función hash. Por defecto usa SHA-1, pero puede configurarse con cualquier método de `hashlib` [`src/itsdangerous/signer.py:48-64`](../../src/itsdangerous/signer.py#L48-L64). Es el algoritmo predeterminado.

- **NoneAlgorithm**: Algoritmo que no realiza firma alguna, retorna una firma vacía [`src/itsdangerous/signer.py:31-37`](../../src/itsdangerous/signer.py#L31-L37). Útil para casos donde la firma no es necesaria.

Puedes especificar un algoritmo personalizado al construir el `Signer` [`src/itsdangerous/signer.py:98-100`](../../src/itsdangerous/signer.py#L98-L100).

## Derivación de claves

El método `derive_key()` [`src/itsdangerous/signer.py:182-213`](../../src/itsdangerous/signer.py#L182-L213) transforma la clave secreta y la sal en una clave de firma. Soporta cuatro estrategias:

- **concat**: `hash(sal + clave_secreta)`
- **django-concat**: `hash(sal + b"signer" + clave_secreta)` — estrategia predeterminada
- **hmac**: `HMAC(clave_secreta, sal)`
- **none**: Usa la clave secreta sin derivación

La derivación se realiza automáticamente durante la firma y verificación.

## Rotación de claves

El `Signer` soporta rotación de claves pasando una lista en lugar de una clave individual [[cite:src/itsdangerous/signer.py:85-86, 138-143]]:

```python
signer = Signer(["old-key", "new-key"])
# Firma con new-key (la última)
signed = signer.sign("data")
# Verifica contra ambas claves (en orden inverso)
signer.unsign(signed)
```

Esto permite actualizar la clave secreta sin invalidar las firmas antiguas.

## Métodos principales

**`sign(value)`** [`src/itsdangerous/signer.py:222-225`](../../src/itsdangerous/signer.py#L222-L225)  
Firma el valor dado y retorna el dato con su firma adjunta.

**`unsign(signed_value)`** [`src/itsdangerous/signer.py:244-256`](../../src/itsdangerous/signer.py#L244-L256)  
Verifica la firma de un dato firmado. Si es válida, retorna el dato original. Si no, lanza `BadSignature` con el payload que falló.

**`verify_signature(value, sig)`** [`src/itsdangerous/signer.py:227-242`](../../src/itsdangerous/signer.py#L227-L242)  
Verifica que una firma es válida para un valor, sin separar. Retorna `True` o `False`.

**`validate(signed_value)`** [`src/itsdangerous/signer.py:258-265`](../../src/itsdangerous/signer.py#L258-L265)  
Valida un dato firmado retornando `True` o `False`, sin lanzar excepciones.

**`get_signature(value)`** [`src/itsdangerous/signer.py:215-220`](../../src/itsdangerous/signer.py#L215-L220)  
Retorna solo la firma para un valor, sin el dato original.

## Excepciones

[`src/itsdangerous/exc.py:22-33`](../../src/itsdangerous/exc.py#L22-L33)  
`BadSignature` es lanzada cuando una firma no coincide. Contiene el atributo `payload` con el dato que falló la verificación.

[`src/itsdangerous/exc.py:7-19`](../../src/itsdangerous/exc.py#L7-L19)  
`BadData` es la excepción base para todos los errores del módulo.

## Configuración

Al construir un `Signer`, puedes personalizar [`src/itsdangerous/signer.py:129-173`](../../src/itsdangerous/signer.py#L129-L173):

- **secret_key**: Clave o lista de claves para firmar. Requerido.
- **salt**: Dato extra para distinguir contextos de firma. Predeterminado: `b"itsdangerous.Signer"`
- **sep**: Separador entre dato y firma. Predeterminado: `b"."`
- **key_derivation**: Método de derivación de clave. Predeterminado: `"django-concat"`
- **digest_method**: Función hash para HMAC. Predeterminado: SHA-1
- **algorithm**: Instancia de `SigningAlgorithm` personalizada

El separador no puede contener caracteres base64, letras, dígitos, `-`, `_` o `=` [`src/itsdangerous/signer.py:146-151`](../../src/itsdangerous/signer.py#L146-L151).

## Seguridad

La seguridad depende de:

1. **Clave secreta aleatoria y fuerte**: No debe guardarse en control de versiones ni código fuente [`src/itsdangerous/signer.py:80-83`](../../src/itsdangerous/signer.py#L80-L83).

2. **Salts distintos por contexto**: Diferentes contextos deben usar salts distintos para evitar reutilizar firmas entre contextos.

3. **Comparación segura**: Las firmas se comparan con `hmac.compare_digest()` para prevenir ataques de tiempo [`src/itsdangerous/signer.py:24-28`](../../src/itsdangerous/signer.py#L24-L28).

Ver [Verificación con marca de tiempo](timestamp-verification.md) para añadir validación de antigüedad a las firmas.
