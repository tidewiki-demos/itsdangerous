# Codificación y Compresión de Datos

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Módulo que maneja la conversión entre diferentes representaciones de datos: texto a bytes, bytes a base64 seguro para URL, e conversiones entre enteros y bytes. Estas operaciones son fundamentales para optimizar el tamaño y compatibilidad de los tokens en el sistema.

## Conversión de Cadenas a Bytes

La función `want_bytes()` [`src/itsdangerous/encoding.py:11-17`](../../src/itsdangerous/encoding.py#L11-L17) normaliza la entrada a bytes independientemente de su tipo inicial. Acepta tanto cadenas de texto como bytes ya codificados, con parámetros configurables para la codificación (UTF-8 por defecto) y manejo de errores (modo estricto por defecto).

```python
want_bytes("mañana")      # → b"ma\xc3\xb1ana"
want_bytes(b"tomorrow")   # → b"tomorrow"
```

Esta función actúa como punto de entrada para asegurar que todas las operaciones posteriores trabajan con bytes, evitando errores de tipo.

## Codificación Base64 Segura para URL

La codificación base64 en itsdangerous utiliza la variante segura para URL definida en RFC 4648, que reemplaza caracteres problemáticos en URLs:

- `base64_encode()` [`src/itsdangerous/encoding.py:20-25`](../../src/itsdangerous/encoding.py#L20-L25) codifica cadenas o bytes a base64 URL-safe, eliminando el relleno con `=` al final. Esto reduce el tamaño del resultado final.

- `base64_decode()` [`src/itsdangerous/encoding.py:28-38`](../../src/itsdangerous/encoding.py#L28-L38) realiza la decodificación inversa. Restaura automáticamente el relleno (que puede ser 0-3 caracteres `=`) antes de decodificar, ya que fue removido durante la codificación. Si los datos no son válidos base64, lanza `BadData`.

```python
base64_encode("無限")  # → b"5pil5pil"  (sin "=")
base64_decode(b"5pil5pil")  # → b"\xe6\x99\xa0\xe9\x99\xa0"
```

El alfabeto utilizado es `A-Za-z0-9-_=`, definido en `_base64_alphabet` [`src/itsdangerous/encoding.py:42`](../../src/itsdangerous/encoding.py#L42).

## Conversión Entre Enteros y Bytes

Para serializar identificadores, marcas temporales u otros valores numéricos de forma compacta:

- `int_to_bytes()` [`src/itsdangerous/encoding.py:49-50`](../../src/itsdangerous/encoding.py#L49-L50) convierte un entero a su representación binaria mínima, eliminando bytes nulos iniciales. Un entero 0 produce bytes vacíos.

- `bytes_to_int()` [`src/itsdangerous/encoding.py:53-54`](../../src/itsdangerous/encoding.py#L53-L54) realiza la conversión inversa. Rellena la entrada con bytes nulos a la izquierda hasta 8 bytes (formato big-endian) antes de desempaquetarla.

```python
int_to_bytes(192)  # → b"\xc0"
bytes_to_int(b"\xc0")  # → 192
```

Internamente, ambas funciones utilizan `struct.Struct(">Q")` [`src/itsdangerous/encoding.py:44`](../../src/itsdangerous/encoding.py#L44) para manejar enteros de 64 bits en formato big-endian.

## Relaciones con Otros Módulos

- [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) utiliza estas funciones para empaquetar datos antes de firmarlos.
- [Codificación Segura para URL](url-safe.md) se basa en `base64_encode()` para generar tokens sin caracteres problemáticos.
- [Gestión de Errores](exceptions.md) documenta la excepción `BadData` lanzada en decodificaciones fallidas.

## Pruebas

El módulo de pruebas [`tests/test_itsdangerous/test_encoding.py`](../../tests/test_itsdangerous/test_encoding.py) cubre:

- `want_bytes()` con texto en múltiples idiomas (incluyendo caracteres no-ASCII).
- Ciclos completos de codificación/decodificación base64 [`tests/test_itsdangerous/test_encoding.py:17-22`](../../tests/test_itsdangerous/test_encoding.py#L17-L22).
- Casos de error en base64 inválido [`tests/test_itsdangerous/test_encoding.py:25-27`](../../tests/test_itsdangerous/test_encoding.py#L25-L27).
- Valores extremos en conversiones entero-bytes, incluyendo 0 y el máximo entero de 64 bits [`tests/test_itsdangerous/test_encoding.py:30-36`](../../tests/test_itsdangerous/test_encoding.py#L30-L36).
