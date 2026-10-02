# Guía de Serialización

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

La serialización es el proceso de convertir objetos Python complejos en formato de bytes o strings que puedan ser almacenados, transmitidos o verificados. Esta guía explica cómo serializar y deserializar objetos de forma segura usando el módulo `itsdangerous.serializer`.

## Conceptos Básicos

A diferencia del [Firmante: Base de Criptografía](core-signer.md), que solo firma bytes, la clase `Serializer` proporciona una interfaz `dumps`/`loads` similar al módulo `json` de Python. Esto permite serializar objetos a una cadena y luego firmar ese resultado, garantizando que el contenido no ha sido modificado.

## Serializar Datos

Para serializar y firmar datos, usa el método [`docs/serializer.rst:11`](../../docs/serializer.rst#L11) `dumps`:

```python
from itsdangerous.serializer import Serializer

s = Serializer("secret-key")
resultado = s.dumps([1, 2, 3, 4])
# b'[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo'
```

El método retorna una cadena de bytes que contiene los datos serializados en JSON seguidos de la firma, separados por un punto [`docs/serializer.rst:28-29`](../../docs/serializer.rst#L28-L29).

## Deserializar y Verificar

Para verificar la firma y deserializar los datos, usa [`docs/serializer.rst:20-21`](../../docs/serializer.rst#L20-L21) el método `loads`:

```python
s.loads('[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo')
# [1, 2, 3, 4]
```

Si la firma no es válida, se lanzará una excepción indicando que los datos han sido manipulados.

## Manejo de Errores en Deserialización

### Inspeccionar la Carga Útil con Atributos de Excepción

Cuando la verificación de firma falla, [`docs/serializer.rst:39-42`](../../docs/serializer.rst#L39-L42) las excepciones contienen atributos útiles que permiten inspeccionar la carga útil. Esto debe hacerse con cuidado, ya que sabes que alguien modificó los datos:

```python
from itsdangerous.serializer import Serializer
from itsdangerous.exc import BadSignature, BadData

s = Serializer("secret-key")
decoded_payload = None

try:
    decoded_payload = s.loads(data)
    # La carga está decodificada y es segura
except BadSignature as e:
    if e.payload is not None:
        try:
            decoded_payload = s.load_payload(e.payload)
        except BadData:
            pass
        # La carga está decodificada pero NO ES SEGURA porque
        # alguien alteró la firma. El paso de decodificación
        # (load_payload) es explícito porque puede ser inseguro
        # deserializar la carga (por ejemplo, con pickle)
```

### Método `loads_unsafe`

[`docs/serializer.rst:67-74`](../../docs/serializer.rst#L67-L74) Si prefieres no inspeccionar atributos para determinar qué falló exactamente, puedes usar `loads_unsafe`:

```python
sig_okay, payload = s.loads_unsafe(data)
```

Este método retorna una tupla donde el primer elemento es un booleano que indica si la firma fue correcta, y el segundo elemento es la carga útil decodificada (independientemente de la validez de la firma).

## Firmantes Alternativos (Fallback)

Puedes actualizar los parámetros de firma sin invalidar las firmas existentes inmediatamente. [`docs/serializer.rst:77-94`](../../docs/serializer.rst#L77-L94) La lista `fallback_signers` se intenta si la verificación de firma con el firmante actual falla. Cada elemento de la lista puede ser:

- Un diccionario de argumentos `signer_kwargs` para instanciar la clase `signer`
- Una instancia o clase de `Signer`
- Una tupla de `(signer_class, signer_kwargs)`

Por ejemplo, para migrar de SHA-1 a SHA-512 manteniendo compatibilidad con firmas antiguas:

```python
import hashlib
from itsdangerous.serializer import Serializer

s = Serializer(
    "secret-key",
    signer_kwargs={"digest_method": hashlib.sha512},
    fallback_signers=[{"digest_method": hashlib.sha1}]
)
```

Las nuevas firmas usarán SHA-512, pero las antiguas con SHA-1 seguirán siendo válidas.

## Formatos de Serialización Especializados

- Para serializar a un formato seguro para URLs, consulta [Codificación Segura para URL](url-safe.md)
- Para incluir marca temporal y validar la antigüedad de las firmas, consulta [Validación con Marca Temporal](timestamp.md)
- Para detalles sobre cómo se serializa a JSON internamente, consulta [Manejo de JSON](json-handling.md)

## Véase También

- [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) — Referencia técnica completa
- [Gestión de Errores](exceptions.md) — Tipos de excepciones y cómo manejarlas
- [Conceptos Fundamentales](concepts.md) — Conceptos subyacentes
- [Referencia de API Pública](api-reference.md) — API completa
