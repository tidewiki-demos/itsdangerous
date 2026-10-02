# Conceptos Fundamentales

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

ItsDangerous es una biblioteca que proporciona dos mecanismos complementarios para proteger datos: firma criptográfica y serialización segura. Esta página introduce los conceptos clave que subyacen a toda la biblioteca.

## Firmante vs Serializador

ItsDangerous ofrece dos niveles de manejo de datos [`docs/concepts.rst:8-11`](../../docs/concepts.rst#L8-L11). El [Firmante (Signer)](firmante.md) es el sistema básico que firma un valor `bytes` basado en parámetros de firma específicos. El [Serializador](serializador.md) envuelve un firmante para permitir serializar y firmar otros tipos de datos además de `bytes`.

En la mayoría de los casos, querrás usar un serializador en lugar de un firmante directamente. El serializador te permite configurar los parámetros de firma y, incluso, proporcionar firmantes alternativos para actualizar tokens antiguos a nuevos parámetros [`docs/concepts.rst:13-15`](../../docs/concepts.rst#L13-L15).

## La Clave Secreta

Las firmas están protegidas por la `secret_key`. Típicamente, una clave secreta se usa con todos los firmantes, y el salt se utiliza para distinguir diferentes contextos [`docs/concepts.rst:23`](../../docs/concepts.rst#L23). Cambiar la clave secreta invalidará los tokens existentes.

La clave secreta debe ser una cadena larga y aleatoria de bytes. Este valor debe mantenerse en secreto y no debe guardarse en el código fuente ni confirmarse en el control de versiones. Si un atacante aprende la clave secreta, puede cambiar y refirmar datos para que se vean válidos [`docs/concepts.rst:26-29`](../../docs/concepts.rst#L26-L29).

Una forma de mantener la clave secreta separada es leerla de una variable de entorno [`docs/concepts.rst:31-35`](../../docs/concepts.rst#L31-L35):

```python
import os
from itsdangerous.serializer import Serializer
SECRET_KEY = os.environ.get("SECRET_KEY")
s = Serializer(SECRET_KEY)
```

Puedes generar una clave usando `os.urandom` [`docs/concepts.rst:49`](../../docs/concepts.rst#L49):

```
$ python3 -c 'import os; print(os.urandom(16).hex())'
```

## El Salt

El salt se combina con la clave secreta para derivar una clave única que distingue diferentes contextos [`docs/concepts.rst:59-62`](../../docs/concepts.rst#L59-L62). A diferencia de la clave secreta, el salt no tiene que ser aleatorio y puede guardarse en el código. Solo debe ser único entre contextos, no privado.

Por ejemplo, si deseas enviar enlaces de activación de correo electrónico para activar cuentas de usuario y enlaces de actualización para pasar a cuentas de pago, si solo firmas el ID de usuario sin usar diferentes salts, un usuario podría reutilizar el token del enlace de activación para actualizar la cuenta. Si usas diferentes salts, las firmas serán diferentes y no serán válidas en el otro contexto [`docs/concepts.rst:64-69`](../../docs/concepts.rst#L64-L69).

[`docs/concepts.rst:71-81`](../../docs/concepts.rst#L71-L81) Dos serializadores con salts distintos producen firmas diferentes para los mismos datos:

```python
from itsdangerous.url_safe import URLSafeSerializer

s1 = URLSafeSerializer("secret-key", salt="activate")
s1.dumps(42)
# 'NDI.MHQqszw6Wc81wOBQszCrEE_RlzY'

s2 = URLSafeSerializer("secret-key", salt="upgrade")
s2.dumps(42)
# 'NDI.c0MpsD6gzpilOAeUPra3NShPXsE'
```

El segundo serializador no puede cargar datos firmados con el primero porque los salts difieren [`docs/concepts.rst:83-84`](../../docs/concepts.rst#L83-L84). Solo el serializador con el mismo salt puede cargar los datos [`docs/concepts.rst:93-98`](../../docs/concepts.rst#L93-L98).

## Rotación de Claves

La rotación de claves proporciona una capa adicional de mitigación contra un atacante que descubre una clave secreta [`docs/concepts.rst:104-110`](../../docs/concepts.rst#L104-L110). Un sistema de rotación mantiene una lista de claves válidas, generando una nueva clave y eliminando la más antigua periódicamente.

En lugar de pasar una sola clave, puedes pasar una lista de claves, de más antigua a más nueva [`docs/concepts.rst:116-119`](../../docs/concepts.rst#L116-L119). Al firmar, se utilizará la última clave (la más nueva), y al validar, se intentará cada clave de más nueva a más antigua antes de lanzar un error de validación:

```python
SECRET_KEYS = ["2b9cd98e", "169d7886", "b6af09f5"]

# firmar algunos datos con la clave más reciente
s = Serializer(SECRET_KEYS)
t = s.dumps({"id": 42})

# rotar una nueva clave y eliminar la más antigua
SECRET_KEYS.append("cf9b3588")
del SECRET_KEYS[0]

s = Serializer(SECRET_KEYS)
s.loads(t)  # válido aunque fue firmado con una clave anterior
```

## Seguridad del Método de Resumen

Un firmante se configura con un `digest_method`, una función hash que se utiliza como paso intermedio al generar la firma HMAC [`docs/concepts.rst:140-141`](../../docs/concepts.rst#L140-L141). El método predeterminado es `hashlib.sha1`. Ocasionalmente, los usuarios expresan preocupación por este predeterminado porque han escuchado sobre colisiones de hash con SHA-1.

Cuando se utiliza como paso intermedio e iterado en HMAC, SHA-1 no es inseguro [`docs/concepts.rst:146-148`](../../docs/concepts.rst#L146-L148). De hecho, incluso MD5 sigue siendo seguro en HMAC. La seguridad del hash por sí solo no se aplica cuando se usa en HMAC.

Si un proyecto considera SHA-1 un riesgo de todas formas, puede configurar el firmante con un método de resumen diferente, como `hashlib.sha512` [`docs/concepts.rst:150-151`](../../docs/concepts.rst#L150-L151). SHA-512 produce un hash más largo, por lo que los tokens ocuparán más espacio, lo que es relevante en cookies y URLs [`docs/concepts.rst:153-154`](../../docs/concepts.rst#L153-L154).
