# Serializador

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Sistema para serializar datos complejos de forma segura, permitiendo convertir objetos Python a tokens firmados y viceversa. El serializador envuelve un [Firmante (Signer)](firmante.md) para habilitar la serialización y firma segura de datos más allá de solo bytes.

## Conceptos básicos

El `Serializer` proporciona una interfaz `dumps`/`loads` similar a la del módulo `json` de Python. Por defecto, utiliza JSON internamente para serializar datos a una cadena, que luego se firma. [`docs/serializer.rst:6-9`](../../docs/serializer.rst#L6-L9)

```python
from itsdangerous.serializer import Serializer
s = Serializer("secret-key")
s.dumps([1, 2, 3, 4])  # b'[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo'
```

Para verificar la integridad, `loads` valida la firma y deserializa los datos: [`docs/serializer.rst:20-26`](../../docs/serializer.rst#L20-L26)

```python
s.loads('[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo')  # [1, 2, 3, 4]
```

## Configuración

### Clave secreta y sal

La clave secreta debe ser una cadena aleatoria de bytes y no debe guardarse en código o control de versiones. Se pueden usar diferentes sales para distinguir firmas en contextos diferentes. Véase [Conceptos Fundamentales](conceptos-fundamentales.md) para información sobre seguridad. [`src/itsdangerous/serializer.py:49-52`](../../src/itsdangerous/serializer.py#L49-L52)

### Serializador personalizado

Por defecto, se usa JSON internamente, pero esto se puede cambiar mediante subclasificación. [`docs/serializer.rst:28-29`](../../docs/serializer.rst#L28-L29) El serializador personalizado debe proporcionar métodos `dumps` y `loads`. [`src/itsdangerous/serializer.py:58-60`](../../src/itsdangerous/serializer.py#L58-L60)

### Rotación de claves

Se puede pasar una lista de claves al parámetro `secret_key`, de más antigua a más nueva, para soportar rotación de claves. La clave más nueva (última) se usa para firmar, mientras que todas las claves se pueden usar para verificar. [[cite:src/itsdangerous/serializer.py:54-56,203-208]]

## Manejo de fallos

### BadSignature y payload

Cuando la validación de firma falla, se lanza `BadSignature`. Esta excepción tiene atributos útiles que permiten inspeccionar el payload, útil para depuración. [`docs/serializer.rst:36-43`](../../docs/serializer.rst#L36-L43)

```python
from itsdangerous.serializer import Serializer
from itsdangerous.exc import BadSignature, BadData

s = Serializer("secret-key")
try:
    decoded_payload = s.loads(data)
except BadSignature as e:
    if e.payload is not None:
        try:
            decoded_payload = s.load_payload(e.payload)
        except BadData:
            pass
```

### loads_unsafe

Si no se desea inspeccionar atributos para determinar qué salió mal, se puede usar `loads_unsafe`: [`docs/serializer.rst:67-74`](../../docs/serializer.rst#L67-L74)

```python
sig_okay, payload = s.loads_unsafe(data)
```

Retorna una tupla donde el primer elemento es un booleano que indica si la firma es válida. Esta función nunca falla, pero no debe usarse con serializadores inseguros como pickle. [`src/itsdangerous/serializer.py:349-365`](../../src/itsdangerous/serializer.py#L349-L365)

## Firmantes de respaldo (Fallback Signers)

Es posible actualizar los parámetros de firma sin invalidar inmediatamente las firmas existentes. Por ejemplo, cambiar el método de hash: las nuevas firmas usan el nuevo método, pero las antiguas siguen siendo válidas. [`docs/serializer.rst:77-83`](../../docs/serializer.rst#L77-L83)

Se puede proporcionar una lista de `fallback_signers`. Cada elemento puede ser: [`docs/serializer.rst:86-94`](../../docs/serializer.rst#L86-L94)

- Un dict de `signer_kwargs` para instanciar la clase `signer`
- Una clase `Signer` a instanciar con `secret_key`, `salt` y `signer_kwargs`
- Una tupla de `(signer_class, signer_kwargs)`

Ejemplo: serializador que firma con SHA-512 pero puede verificar con SHA-512 o SHA-1: [`docs/serializer.rst:99-104`](../../docs/serializer.rst#L99-L104)

```python
s = Serializer(
    signer_kwargs={"digest_method": hashlib.sha512},
    fallback_signers=[{"digest_method": hashlib.sha1}]
)
```

## Métodos principales

[`src/itsdangerous/serializer.py:309-320`](../../src/itsdangerous/serializer.py#L309-L320)

- **`dumps(obj, salt=None)`**: Retorna una cadena firmada serializada. El valor de retorno puede ser bytes o string según el formato del serializador interno.

- **`loads(s, salt=None)`**: Inverso de `dumps`. Lanza `BadSignature` si la validación de firma falla. Itera sobre todos los firmantes (incluyendo fallback_signers) hasta encontrar uno válido. [`src/itsdangerous/serializer.py:328-343`](../../src/itsdangerous/serializer.py#L328-L343)

- **`dump(obj, f, salt=None)`**: Como `dumps` pero escribe en un archivo.

- **`load(f, salt=None)`**: Como `loads` pero lee de un archivo.

- **`load_payload(payload, serializer=None)`**: Carga el objeto codificado. Lanza `BadPayload` si el payload no es válido. [`src/itsdangerous/serializer.py:243-269`](../../src/itsdangerous/serializer.py#L243-L269)

- **`dump_payload(obj)`**: Serializa un objeto. El valor de retorno siempre es bytes; si el serializador interno retorna texto, se codifica como UTF-8. [`src/itsdangerous/serializer.py:271-276`](../../src/itsdangerous/serializer.py#L271-L276)

- **`make_signer(salt=None)`**: Crea una nueva instancia del firmante. [`src/itsdangerous/serializer.py:278-285`](../../src/itsdangerous/serializer.py#L278-L285)

- **`iter_unsigners(salt=None)`**: Itera sobre todos los firmantes a probar para verificar: primero el firmante configurado, luego cada uno especificado en `fallback_signers`. [`src/itsdangerous/serializer.py:287-307`](../../src/itsdangerous/serializer.py#L287-L307)

## Codificación

El serializador internamente detecta si el serializador produce texto o bytes mediante `is_text_serializer`. Para serializadores de texto, el payload se decodifica como UTF-8 antes de pasar al serializador. [[cite:src/itsdangerous/serializer.py:33-37,220,260-263]]

Véase [Codificación y Excepciones](codificacion-y-excepciones.md) para más detalles sobre el manejo de encodings.

## Variantes

- **[Serializador con Tiempo](firmante-con-tiempo.md)**: Registra y valida la edad de la firma.
- **[Codificación URL Segura](url-segura.md)**: Serializa a un formato seguro para URLs.
