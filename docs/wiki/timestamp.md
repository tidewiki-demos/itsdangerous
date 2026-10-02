# Validación con Marca Temporal

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Añade capacidades de marca temporal a los firmantes, permitiendo verificar que los datos fueron firmados dentro de un período específico. Las marcas temporales se codifican como parte de la firma y se validan durante la verificación.

## Componentes principales

### TimestampSigner

[`src/itsdangerous/timed.py:22-27`](../../src/itsdangerous/timed.py#L22-L27) La clase `TimestampSigner` extiende [Signer](core-signer.md) para incluir información de tiempo en cada firma. El método `unsign` puede lanzar `SignatureExpired` si la firma ha expirado.

**Obtención de marca temporal:** [`src/itsdangerous/timed.py:29-33`](../../src/itsdangerous/timed.py#L29-L33) El método `get_timestamp()` retorna el timestamp actual en segundos como entero. Puede ser sobrescrito para propósitos de testing.

**Conversión a datetime:** [`src/itsdangerous/timed.py:35-43`](../../src/itsdangerous/timed.py#L35-L43) El método `timestamp_to_datetime()` convierte un timestamp a un objeto `datetime` en UTC, retornando un datetime consciente de zona horaria.

**Firma con timestamp:** [`src/itsdangerous/timed.py:45-51`](../../src/itsdangerous/timed.py#L45-L51) El método `sign()` toma el valor a firmar, obtiene el timestamp actual, lo codifica en base64, y lo incluye en el valor antes de calcular la firma. El formato es: `valor.timestamp_codificado.firma`.

**Verificación con validación temporal:** [`src/itsdangerous/timed.py:72-158`](../../src/itsdangerous/timed.py#L72-L158) El método `unsign()` verifica la firma y valida la antigüedad usando el parámetro `max_age` (en segundos):

- Si `max_age` es especificado, verifica que la edad de la firma no exceda ese período [`src/itsdangerous/timed.py:138-146`](../../src/itsdangerous/timed.py#L138-L146).
- También detecta firmas con timestamp futuro (edad negativa) [`src/itsdangerous/timed.py:148-153`](../../src/itsdangerous/timed.py#L148-L153).
- Si `return_timestamp=True`, retorna una tupla `(valor, datetime)` [`src/itsdangerous/timed.py:155-156`](../../src/itsdangerous/timed.py#L155-L156).

**Validación simple:** [`src/itsdangerous/timed.py:160-167`](../../src/itsdangerous/timed.py#L160-L167) El método `validate()` retorna `True` si la firma es válida, sin excepciones.

### TimedSerializer

[`src/itsdangerous/timed.py:170-176`](../../src/itsdangerous/timed.py#L170-L176) La clase `TimedSerializer` utiliza `TimestampSigner` por defecto en lugar del `Signer` ordinario, extendiendo la capacidad de marca temporal a la [serialización](serializer.md) de estructuras de datos.

**Carga con validación temporal:** [`src/itsdangerous/timed.py:185-220`](../../src/itsdangerous/timed.py#L185-L220) El método `loads()` deserializa datos y puede validar su antigüedad:

- Acepta el parámetro `max_age` para rechazar datos más antiguos que el período especificado.
- Si `return_timestamp=True`, retorna una tupla `(payload, datetime)`.
- Lanza `SignatureExpired` si la firma ha expirado, sin intentar otros firmantes.

**Carga insegura:** [`src/itsdangerous/timed.py:222-228`](../../src/itsdangerous/timed.py#L222-L228) El método `loads_unsafe()` permite recuperar datos incluso si la validación falla, útil para análisis forense.

## Manejo de errores

El proceso de verificación maneja varios casos de error relacionados con marcas temporales:

- **Timestamp faltante:** [`src/itsdangerous/timed.py:102-106`](../../src/itsdangerous/timed.py#L102-L106) Si no hay timestamp en el resultado, se lanza `BadTimeSignature`.
- **Timestamp malformado:** [`src/itsdangerous/timed.py:112-135`](../../src/itsdangerous/timed.py#L112-L135) Si la decodificación del timestamp falla, se lanza `BadTimeSignature`. Se intenta convertir a `datetime` para incluir información de fecha si es posible [`src/itsdangerous/timed.py:121-128`](../../src/itsdangerous/timed.py#L121-L128).
- **Firma inválida:** [`src/itsdangerous/timed.py:119-130`](../../src/itsdangerous/timed.py#L119-L130) Si la firma es inválida pero el timestamp se puede decodificar, se incluye la fecha firmada en la excepción `BadTimeSignature`.

Estas excepciones se documentan en [Gestión de Errores](exceptions.md).

## Codificación

La marca temporal se codifica usando [`src/itsdangerous/timed.py:9-12`](../../src/itsdangerous/timed.py#L9-L12) funciones de [Codificación Segura para URL](encoding.md):

- El timestamp en segundos se convierte a bytes entero [`src/itsdangerous/timed.py:48`](../../src/itsdangerous/timed.py#L48) usando `int_to_bytes()`.
- Se codifica en base64 para ser seguro para URLs [`src/itsdangerous/timed.py:48`](../../src/itsdangerous/timed.py#L48) usando `base64_encode()`.
- Durante la verificación se decodifica usando `base64_decode()` y `bytes_to_int()` [`src/itsdangerous/timed.py:113`](../../src/itsdangerous/timed.py#L113).

## Casos de uso

- **Tokens de sesión:** Verificar que un token se emitió recientemente.
- **Links de confirmación:** Asegurar que un link de activación no sea antiguo.
- **Datos firmados con restricción temporal:** Permitir solo firmas recientes en operaciones sensibles.
- **Auditoría:** Retornar la fecha exacta en que se firmaron los datos.

Para ejemplos de uso, véase [Guía de Marcas Temporales](timestamp-guide.md).
