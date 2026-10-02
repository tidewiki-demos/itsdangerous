# Firmante con Tiempo

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Extensión del [Firmante](firmante.md) que añade marcas de tiempo a los tokens para verificar su antigüedad y validez temporal. Permite expirar firmas después de un período especificado.

## Concepto

[`src/itsdangerous/timed.py:22-27`](../../src/itsdangerous/timed.py#L22-L27) `TimestampSigner` extiende `Signer` registrando el momento en que se firma un valor. Al verificar la firma con el método `unsign`, se puede validar que la firma no sea más antigua que una edad máxima especificada.

El formato de una firma con marca de tiempo incluye tres partes separadas por un delimitador:
1. El valor original
2. La marca de tiempo codificada en base64
3. La firma criptográfica

## TimestampSigner

[`src/itsdangerous/timed.py:22-168`](../../src/itsdangerous/timed.py#L22-L168)

### Métodos principales

**`sign(value)`**: [`src/itsdangerous/timed.py:45-51`](../../src/itsdangerous/timed.py#L45-L51) Firma el valor adjuntando información de tiempo. Convierte el timestamp actual a bytes, lo codifica en base64, y lo añade al valor antes de generar la firma.

**`unsign(signed_value, max_age, return_timestamp)`**: [`src/itsdangerous/timed.py:72-158`](../../src/itsdangerous/timed.py#L72-L158) Verifica la firma y extrae el valor y timestamp. Si se proporciona `max_age`, valida que la firma no sea más antigua que esa cantidad de segundos. Si `return_timestamp` es `True`, retorna una tupla `(valor, datetime)` con el timestamp convertido a objeto `datetime` en UTC.

**`get_timestamp()`**: [`src/itsdangerous/timed.py:29-33`](../../src/itsdangerous/timed.py#L29-L33) Retorna el timestamp actual en segundos Unix como entero.

**`timestamp_to_datetime(ts)`**: [`src/itsdangerous/timed.py:35-43`](../../src/itsdangerous/timed.py#L35-L43) Convierte un timestamp a un objeto `datetime` consciente de zona horaria en UTC.

**`validate(signed_value, max_age)`**: [`src/itsdangerous/timed.py:160-167`](../../src/itsdangerous/timed.py#L160-L167) Método de conveniencia que retorna `True` si la firma es válida, `False` en caso contrario.

### Manejo de errores

[`src/itsdangerous/timed.py:88-153`](../../src/itsdangerous/timed.py#L88-L153) El método `unsign` puede lanzar dos tipos de excepciones:

- `BadTimeSignature`: Si falta o está malformada la marca de tiempo, o si la firma criptográfica es inválida. En estos casos incluye `date_signed` si se pudo extraer el timestamp.
- `SignatureExpired`: Si el timestamp es más antiguo que `max_age` o si es del futuro (edad negativa).

## TimedSerializer

[`src/itsdangerous/timed.py:170-228`](../../src/itsdangerous/timed.py#L170-L228) Versión serializada que usa `TimestampSigner` internamente en lugar del `Signer` estándar.

### Métodos principales

**`loads(s, max_age, return_timestamp, salt)`**: [`src/itsdangerous/timed.py:185-220`](../../src/itsdangerous/timed.py#L185-L220) Deserializa y verifica una firma con timestamp. Acepta un parámetro `max_age` para validar antigüedad. Si `return_timestamp` es `True`, retorna una tupla `(payload, datetime)`.

**`loads_unsafe(s, max_age, salt)`**: [`src/itsdangerous/timed.py:222-228`](../../src/itsdangerous/timed.py#L222-L228) Variante insegura que retorna `(is_valid, payload)` sin lanzar excepciones.

## Casos de uso

[`tests/test_itsdangerous/test_timed.py:34-43`](../../tests/test_itsdangerous/test_timed.py#L34-L43) Validación de antigüedad máxima: se puede firmar un valor y verificarlo solo si no ha expirado según `max_age`.

[`tests/test_itsdangerous/test_timed.py:45-47`](../../tests/test_itsdangerous/test_timed.py#L45-L47) Recuperación del timestamp: usando `return_timestamp=True` se obtiene tanto el valor como el momento exacto en que se firmó.

[`tests/test_itsdangerous/test_timed.py:49-76`](../../tests/test_itsdangerous/test_timed.py#L49-L76) Manejo de marcas de tiempo malformadas o faltantes: detecta intentos de usar firmas normales con un desserializador basado en tiempo.

## Decisiones

La marca de tiempo se codifica en base64 (en lugar de guardarse como texto plano) para mantener consistencia con la [Codificación y Excepciones](codificacion-y-excepciones.md) y asegurar compatibilidad con firmas de bytes. El timestamp se almacena como entero Unix por su compacidad y facilidad de comparación temporal.
