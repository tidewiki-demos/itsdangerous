# Verificación con marca de tiempo

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Extensión que añade y verifica automáticamente marcas de tiempo en los tokens para controlar su validez temporal. Permite firmar datos con información de cuándo se realizó la firma y validar que no hayan expirado.

## Concepto

[`src/itsdangerous/timed.py:22-27`](../../src/itsdangerous/timed.py#L22-L27) La clase `TimestampSigner` extiende la funcionalidad de firma regular añadiendo registro automático del momento de la firma. Al verificar (unsign), es posible validar que la firma no sea más antigua que una edad máxima especificada.

El flujo es:
1. Al firmar, se captura el timestamp actual, se codifica en base64 y se incluye en el valor firmado
2. Al verificar, se extrae el timestamp, se decodifica y se compara con la edad máxima permitida
3. Si la firma es demasiado antigua, se lanza `SignatureExpired`

## Clases principales

### TimestampSigner

[`src/itsdangerous/timed.py:22-168`](../../src/itsdangerous/timed.py#L22-L168) Implementa la firma con marca de tiempo:

- **`get_timestamp()`** [`src/itsdangerous/timed.py:29-33`](../../src/itsdangerous/timed.py#L29-L33): Retorna el timestamp actual como entero (segundos desde epoch).

- **`timestamp_to_datetime(ts)`** [`src/itsdangerous/timed.py:35-43`](../../src/itsdangerous/timed.py#L35-L43): Convierte un timestamp entero a un objeto `datetime` consciente de zona horaria en UTC.

- **`sign(value)`** [`src/itsdangerous/timed.py:45-51`](../../src/itsdangerous/timed.py#L45-L51): Firma el valor añadiendo el timestamp codificado en base64. El resultado tiene la estructura: `valor.timestamp_base64.firma`.

- **`unsign(signed_value, max_age=None, return_timestamp=False)`** [`src/itsdangerous/timed.py:72-158`](../../src/itsdangerous/timed.py#L72-L158): Verifica la firma y valida el timestamp. Parámetros:
  - `max_age`: edad máxima permitida en segundos. Si es `None`, no se valida la edad.
  - `return_timestamp`: si es `True`, retorna una tupla `(valor, datetime)` en lugar de solo el valor.
  
  Lanza `SignatureExpired` si el timestamp es más antiguo que `max_age` o si es negativo (futuro). Lanza `BadTimeSignature` si el timestamp no está presente o está malformado.

- **`validate(signed_value, max_age=None)`** [`src/itsdangerous/timed.py:160-167`](../../src/itsdangerous/timed.py#L160-L167): Solo valida sin lanzar excepciones. Retorna `True` si la firma es válida.

### TimedSerializer

[`src/itsdangerous/timed.py:170-228`](../../src/itsdangerous/timed.py#L170-L228) Serializa datos con timestamps. Extiende [Serialización y compresión](serializer.md) usando `TimestampSigner` por defecto.

- **`loads(s, max_age=None, return_timestamp=False, salt=None)`** [`src/itsdangerous/timed.py:185-212`](../../src/itsdangerous/timed.py#L185-L212): Deserializa y verifica la firma con timestamp. Si algún firmante firma exitosamente pero la firma está expirada, lanza `SignatureExpired` sin intentar los siguientes firmantes.

- **`loads_unsafe(s, max_age=None, salt=None)`** [`src/itsdangerous/timed.py:222-228`](../../src/itsdangerous/timed.py#L222-L228): Deserializa sin validar, pasando `max_age` para casos especiales.

## Manejo de errores

[`src/itsdangerous/timed.py:88-158`](../../src/itsdangerous/timed.py#L88-L158) El método `unsign` maneja varios casos de error:

- Si la firma no es válida pero hay timestamp decodificable, incluye la información del timestamp en la excepción `BadTimeSignature`
- Si no hay timestamp en el resultado, lanza `BadTimeSignature("timestamp missing")`
- Si el timestamp está malformado o no se puede convertir a `datetime`, lanza `BadTimeSignature("Malformed timestamp")`
- [`src/itsdangerous/timed.py:138-153`](../../src/itsdangerous/timed.py#L138-L153) Si la edad es mayor que `max_age` o es negativa (timestamp futuro), lanza `SignatureExpired` con el mensaje especificando la diferencia

## Uso

```python
from itsdangerous import TimestampSigner

s = TimestampSigner('secret-key')
signed = s.sign('foo')

# Verificar dentro de la edad máxima
s.unsign(signed, max_age=5)  # OK si menos de 5 segundos

# Verificar después de expiración
s.unsign(signed, max_age=5)  # Lanza SignatureExpired si pasaron más de 5 segundos

# Obtener el timestamp de la firma
value, timestamp = s.unsign(signed, return_timestamp=True)
```

[`tests/test_itsdangerous/test_timed.py:34-43`](../../tests/test_itsdangerous/test_timed.py#L34-L43) Las pruebas demuestran validación de edad máxima con control de tiempo congelado.

[`tests/test_itsdangerous/test_timed.py:45-47`](../../tests/test_itsdangerous/test_timed.py#L45-L47) Se puede retornar el timestamp junto con el valor.

## Integración

La verificación con marca de tiempo se integra con:

- [Firma de datos](signer.md): `TimestampSigner` extiende `Signer` reutilizando su lógica de firma
- [Serialización y compresión](serializer.md): `TimedSerializer` usa `TimestampSigner` como firmante por defecto
- [Codificación segura para URL](url-safe-encoding.md): Las funciones de codificación base64 se usan para comprimir el timestamp
