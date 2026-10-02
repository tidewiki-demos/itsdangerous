# Codificación URL Segura

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Esta página describe las variantes de serialización diseñadas para transmitir datos de forma segura en URLs y cookies, eliminando caracteres problemáticos mediante codificación base64 y compresión opcional.

## Descripción General

Cuando datos firmados deben transmitirse en contextos donde solo está disponible un conjunto limitado de caracteres (como URLs o cookies HTTP), `URLSafeSerializer` y `URLSafeTimedSerializer` proporcionan una alternativa al [Serializador](serializador.md) estándar.

El mecanismo garantiza que la salida contiene únicamente caracteres seguros para URL:
- Letras mayúsculas y minúsculas
- Guiones (`-`) y guiones bajos (`_`)
- Puntos (`.`)

[`src/itsdangerous/url_safe.py:72-76`](../../src/itsdangerous/url_safe.py#L72-L76)
[`src/itsdangerous/url_safe.py:79-83`](../../src/itsdangerous/url_safe.py#L79-L83)

## Funcionamiento

### Codificación del Payload

[`src/itsdangerous/url_safe.py:55-69`](../../src/itsdangerous/url_safe.py#L55-L69)

El proceso de serialización realiza dos transformaciones:

1. **Compresión opcional con zlib**: Si el resultado comprimido es más corto que el original, se comprime el payload. Un punto (`.`) al inicio indica que el dato está comprimido.
2. **Codificación base64**: El resultado se codifica en base64 para garantizar que solo contiene caracteres seguros para URL.

### Decodificación del Payload

[`src/itsdangerous/url_safe.py:23-53`](../../src/itsdangerous/url_safe.py#L23-L53)

Durante la carga, el proceso invierte las transformaciones:

1. Detecta si el payload comienza con `.` (indicador de compresión).
2. Decodifica el contenido de base64.
3. Si estaba comprimido, descomprime con zlib.
4. Pasa el resultado al método `load_payload` de la clase padre para validar la firma.

## Clases Disponibles

### URLSafeSerializer

[`src/itsdangerous/url_safe.py:72-76`](../../src/itsdangerous/url_safe.py#L72-L76)

Funciona como [Serializer](serializador.md) pero produce cadenas seguras para URL. Es la opción básica cuando no se requiere información de tiempo.

**Ejemplo de uso:**

```python
from itsdangerous.url_safe import URLSafeSerializer

s = URLSafeSerializer("secret-key")
resultado = s.dumps([1, 2, 3, 4])
# 'WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo'

datos = s.loads("WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo")
# [1, 2, 3, 4]
```

### URLSafeTimedSerializer

[`src/itsdangerous/url_safe.py:79-83`](../../src/itsdangerous/url_safe.py#L79-L83)

Combina la codificación URL segura con la capacidad de [Firmante con Tiempo](firmante-con-tiempo.md). Permite establecer y validar ventanas de tiempo para la expiración del dato.

## Codificación Base64 Segura para URL

El módulo utiliza funciones especializadas de codificación `base64_encode` y `base64_decode` que producen caracteres seguros para URL (sin `+`, `/` ni `=` problemáticos). Estas operan sobre los datos procesados por el [serializador JSON compacto](serializador.md).

[`src/itsdangerous/url_safe.py:7-8`](../../src/itsdangerous/url_safe.py#L7-L8)
[`src/itsdangerous/url_safe.py:21`](../../src/itsdangerous/url_safe.py#L21)

## Manejo de Errores

Si durante la decodificación ocurre un error en base64 o zlib, se genera una excepción `BadPayload` que indica la naturaleza del problema. Consulta [Codificación y Excepciones](codificacion-y-excepciones.md) para más detalles.

[`src/itsdangerous/url_safe.py:36-51`](../../src/itsdangerous/url_safe.py#L36-L51)

## Pruebas

Las pruebas verifican que ambas variantes funcionan correctamente con payloads de tamaño variable, incluyendo casos donde la compresión es beneficiosa.

[`tests/test_itsdangerous/test_url_safe.py:11-24`](../../tests/test_itsdangerous/test_url_safe.py#L11-L24)
