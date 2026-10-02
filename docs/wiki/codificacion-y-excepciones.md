# Codificación y Excepciones

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Proporciona utilidades para manejar la codificación de caracteres y define las excepciones personalizadas del sistema. Estos componentes son fundamentales para garantizar que los datos se procesen correctamente y que los errores se reporten de forma significativa.

## Utilidades de Codificación

Las funciones de codificación en [`src/itsdangerous/encoding.py`](../../src/itsdangerous/encoding.py) permiten convertir entre diferentes formatos de datos:

**`want_bytes(s, encoding="utf-8", errors="strict")`** [`src/itsdangerous/encoding.py:11-17`](../../src/itsdangerous/encoding.py#L11-L17)
Convierte una cadena de texto o bytes a bytes. Si la entrada es una cadena (`str`), la codifica con la codificación especificada. Esto es útil cuando se necesita garantizar que los datos están en formato de bytes para operaciones criptográficas.

**`base64_encode(string)`** [`src/itsdangerous/encoding.py:20-25`](../../src/itsdangerous/encoding.py#L20-L25)
Codifica una cadena de texto o bytes en Base64 usando el alfabeto seguro para URLs (`urlsafe_b64encode`). El resultado elimina el relleno con `=` al final, lo que permite que los datos codificados se transmitan sin problemas en URLs. Internamente utiliza `want_bytes` para asegurar que la entrada sea bytes.

**`base64_decode(string)`** [`src/itsdangerous/encoding.py:28-38`](../../src/itsdangerous/encoding.py#L28-L38)
Decodifica una cadena Base64 codificada de forma segura para URLs. La función restaura el relleno necesario [`src/itsdangerous/encoding.py:32-33`](../../src/itsdangerous/encoding.py#L32-L33) y captura errores de decodificación inválida, lanzando `BadData` con un mensaje descriptivo [`src/itsdangerous/encoding.py:37-38`](../../src/itsdangerous/encoding.py#L37-L38).

**Funciones de conversión de enteros**
- `int_to_bytes(num)` [`src/itsdangerous/encoding.py:49-50`](../../src/itsdangerous/encoding.py#L49-L50): Convierte un entero a su representación en bytes, eliminando bytes nulos al inicio.
- `bytes_to_int(bytestr)` [`src/itsdangerous/encoding.py:53-54`](../../src/itsdangerous/encoding.py#L53-L54): Convierte una secuencia de bytes a un entero, rellenando con bytes nulos si es necesario.

Estas funciones utilizan un struct de 64 bits big-endian [`src/itsdangerous/encoding.py:44-46`](../../src/itsdangerous/encoding.py#L44-L46) para garantizar una conversión consistente.

## Jerarquía de Excepciones

El módulo [`src/itsdangerous/exc.py`](../../src/itsdangerous/exc.py) define una jerarquía de excepciones personalizada que hereda de una clase base común:

```
BadData
├── BadSignature
│   ├── BadTimeSignature
│   │   └── SignatureExpired
│   └── BadHeader
└── BadPayload
```

**`BadData`** [`src/itsdangerous/exc.py:7-19`](../../src/itsdangerous/exc.py#L7-L19)
La excepción base para todos los errores de ItsDangerous. Almacena el mensaje de error en el atributo `message` y lo expone a través de `__str__`.

**`BadSignature`** [`src/itsdangerous/exc.py:22-33`](../../src/itsdangerous/exc.py#L22-L33)
Se lanza cuando una firma no coincide con los datos. Incluye el atributo `payload` que contiene los datos que fallaron la verificación, útil para inspeccionar datos incluso si fueron alterados.

**`BadTimeSignature`** [`src/itsdangerous/exc.py:36-57`](../../src/itsdangerous/exc.py#L36-L57)
Subclase de `BadSignature` para firmas basadas en tiempo. Añade el atributo `date_signed` que contiene la fecha en que se creó la firma (con información de zona horaria desde la versión 2.0), permitiendo informar al usuario cuánto tiempo ha estado inactivo un enlace.

**`SignatureExpired`** [`src/itsdangerous/exc.py:60-63`](../../src/itsdangerous/exc.py#L60-L63)
Se lanza cuando una marca de tiempo de firma es más antigua que `max_age`. Es una subclase de `BadTimeSignature`.

**`BadHeader`** [`src/itsdangerous/exc.py:66-89`](../../src/itsdangerous/exc.py#L66-L89)
Se lanza cuando un encabezado firmado es inválido. Solo ocurre para serializadores que tienen un encabezado asociado con la firma. Proporciona:
- `header`: El encabezado real si está disponible pero malformado.
- `original_error`: La excepción original que indica por qué el encabezado no era válido.

**`BadPayload`** [`src/itsdangerous/exc.py:92-106`](../../src/itsdangerous/exc.py#L92-L106)
Se lanza cuando una carga útil (payload) es inválida, ya sea porque se cargó a pesar de una firma inválida o porque hay una discrepancia entre el serializador y deserializador. Incluye `original_error` para almacenar la excepción subyacente.

## Uso en el Sistema

Estas excepciones se utilizan en todo el sistema:
- El [Firmante (Signer)](firmante.md) las lanza durante la verificación de firmas.
- El [Serializador](serializador.md) las utiliza para reportar problemas de serialización.
- El [Firmante con Tiempo](firmante-con-tiempo.md) las emplea para validar marcas de tiempo.

La codificación Base64 es especialmente importante para la [Codificación URL Segura](url-segura.md), garantizando que los datos firmados puedan transmitirse de forma segura en URLs sin caracteres problemáticos.
