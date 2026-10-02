# Codificación Segura para URL

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Especializa los serializadores para generar tokens seguros que pueden incluirse directamente en URLs, evitando caracteres problemáticos mediante codificación base64 y compresión opcional.

## Descripción General

Este módulo proporciona dos clases que extienden los serializadores estándar con capacidades de codificación segura para URL:

- **URLSafeSerializer**: versión segura para URL del [Serializador base](serializer.md)
- **URLSafeTimedSerializer**: versión segura para URL del [Serializador con marca temporal](timestamp.md)

Ambas heredan de `URLSafeSerializerMixin`, que implementa la lógica central de transformación de datos.

## Cómo Funciona

### Serialización (dump_payload)

[`src/itsdangerous/url_safe.py:55-69`](../../src/itsdangerous/url_safe.py#L55-L69)

El proceso de serialización realiza tres operaciones:

1. **Serialización base**: delega al serializador padre para convertir el objeto a JSON usando `_CompactJSON` [`src/itsdangerous/url_safe.py:21`](../../src/itsdangerous/url_safe.py#L21)
2. **Compresión opcional**: intenta comprimir con zlib [`src/itsdangerous/url_safe.py:58`](../../src/itsdangerous/url_safe.py#L58). Si el resultado comprimido es más corto, se utiliza y se marca con un prefijo `b"."` [`src/itsdangerous/url_safe.py:60-67`](../../src/itsdangerous/url_safe.py#L60-L67)
3. **Codificación base64**: transforma el resultado a base64 para garantizar que solo contiene caracteres seguros para URL (letras, `_`, `-` y `.`)

### Deserialización (load_payload)

[`src/itsdangerous/url_safe.py:23-53`](../../src/itsdangerous/url_safe.py#L23-L53)

El proceso inverso:

1. **Detección de compresión**: si el payload comienza con `b"."`, se marca para descomprimir y se elimina este prefijo [`src/itsdangerous/url_safe.py:32-34`](../../src/itsdangerous/url_safe.py#L32-L34)
2. **Decodificación base64**: convierte desde base64 a bytes binarios [`src/itsdangerous/url_safe.py:37`](../../src/itsdangerous/url_safe.py#L37)
3. **Descompresión condicional**: si se detectó compresión, descomprime con zlib [`src/itsdangerous/url_safe.py:44-51`](../../src/itsdangerous/url_safe.py#L44-L51)
4. **Deserialización base**: delega al serializador padre para convertir JSON a objeto

## Manejo de Errores

Ambas operaciones envuelven excepciones en `BadPayload` con mensajes descriptivos:

- Base64: "Could not base64 decode the payload because of an exception" [`src/itsdangerous/url_safe.py:39-42`](../../src/itsdangerous/url_safe.py#L39-L42)
- Zlib: "Could not zlib decompress the payload before decoding the payload" [`src/itsdangerous/url_safe.py:48-51`](../../src/itsdangerous/url_safe.py#L48-L51)

Consulta [Gestión de Errores](exceptions.md) para más detalles.

## Caracteres Seguros para URL

El resultado final contiene solo caracteres seguros para usar en URLs sin codificación adicional: letras mayúsculas y minúsculas, números, `_`, `-` y `.`. Para más información sobre la [Codificación y Compresión de Datos](encoding.md).

## Decisiones de Diseño

- **Compresión automática**: se aplica solo cuando reduce el tamaño, optimizando tokens largos sin penalizar los cortos
- **Prefijo para compresión**: el marcador `b"."` es legible en la base64 resultante y distingue payloads comprimidos sin información extra
- **Serializador compacto**: usa `_CompactJSON` por defecto para minimizar el tamaño antes de comprimir

## Referencias

- [Guía de Tokens Seguros para URL](url-safe-guide.md) — guía de uso
- [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) — serializador base
- [Validación con Marca Temporal](timestamp.md) — versión con timestamping
- [Codificación y Compresión de Datos](encoding.md) — funciones base64 y zlib
