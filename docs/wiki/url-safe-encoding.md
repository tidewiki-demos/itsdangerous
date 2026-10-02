# Codificación segura para URL

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Proporciona funciones para codificar datos en formato seguro para URLs y bases de datos, con soporte automático para caracteres especiales y compresión opcional.

## Funciones de codificación base64

[`src/itsdangerous/encoding.py:20-25`](../../src/itsdangerous/encoding.py#L20-L25) La función `base64_encode()` codifica texto o bytes usando base64 seguro para URLs. Elimina el relleno con `=` para mantener la cadena más corta y compatible con URLs.

[`src/itsdangerous/encoding.py:28-38`](../../src/itsdangerous/encoding.py#L28-L38) La función `base64_decode()` revierte el proceso: decodifica una cadena base64 segura para URLs, restaura el relleno necesario y genera una excepción `BadData` si los datos no son válidos.

[`src/itsdangerous/encoding.py:11-17`](../../src/itsdangerous/encoding.py#L11-L17) La función auxiliar `want_bytes()` convierte texto o bytes a bytes usando la codificación especificada (por defecto UTF-8), permitiendo que las funciones siguientes trabajen uniformemente.

## Serialización segura para URLs

[`src/itsdangerous/url_safe.py:15-19`](../../src/itsdangerous/url_safe.py#L15-L19) La clase `URLSafeSerializerMixin` combina compresión con zlib y codificación base64 para generar cadenas seguras para URLs. Si el resultado comprimido es más corto, usa compresión automáticamente y prepone un punto (`.`) como marcador.

[`src/itsdangerous/url_safe.py:72-76`](../../src/itsdangerous/url_safe.py#L72-L76) `URLSafeSerializer` funciona como un serializador estándar pero genera cadenas compuestas solo por letras (mayúsculas y minúsculas), dígitos, guiones (`-`), guiones bajos (`_`) y puntos (`.`).

[`src/itsdangerous/url_safe.py:79-83`](../../src/itsdangerous/url_safe.py#L79-L83) `URLSafeTimedSerializer` combina serialización segura para URLs con marcas de tiempo, permitiendo verificar que los datos no han expirado.

## Flujo de serialización

[`src/itsdangerous/url_safe.py:55-69`](../../src/itsdangerous/url_safe.py#L55-L69) En la serialización (`dump_payload`):
1. Se serializa el objeto a JSON
2. Se comprime con zlib si el resultado es más corto
3. Se codifica en base64 seguro para URLs
4. Si se comprimió, se prepone un punto como marcador

[`src/itsdangerous/url_safe.py:23-53`](../../src/itsdangerous/url_safe.py#L23-L53) En la deserialización (`load_payload`):
1. Se detecta si hay compresión (marcador inicial `.`)
2. Se decodifica base64
3. Se descomprime si fue necesario
4. Se deserializa el JSON

## Ejemplo de uso

[`docs/url_safe.rst:10-17`](../../docs/url_safe.rst#L10-L17) Un `URLSafeSerializer` transforma datos en cadenas seguras para URLs y las revierte:

```python
from itsdangerous.url_safe import URLSafeSerializer
s = URLSafeSerializer("secret-key")
s.dumps([1, 2, 3, 4])
# 'WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo'
s.loads("WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo")
# [1, 2, 3, 4]
```

El resultado puede incluirse directamente en URLs, parámetros de consulta o cookies sin necesidad de procesamiento adicional.

## Soporte para caracteres especiales

[`tests/test_itsdangerous/test_encoding.py:11-22`](../../tests/test_itsdangerous/test_encoding.py#L11-L22) Las funciones manejan correctamente caracteres especiales y Unicode (como `"mañana"` o `"無限"`), convirtiéndolos internamente a bytes antes de codificar.

[`src/itsdangerous/encoding.py:41-42`](../../src/itsdangerous/encoding.py#L41-L42) El alfabeto utilizado es el estándar de base64 seguro para URLs: letras ASCII (mayúsculas y minúsculas), dígitos, guiones, guiones bajos e iguales.

## Relacionado

- [Serialización y compresión](serializer.md) — Base de la serialización de datos
- [Firma de datos](signer.md) — Integración con firmado de datos
- [Verificación con marca de tiempo](timestamp-verification.md) — Para versionado temporal de datos
