# Serialización y compresión

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

La serialización en itsdangerous permite convertir objetos Python a formato seguro para almacenar o transmitir. [`docs/serializer.rst:6-9`](../../docs/serializer.rst#L6-L9) El módulo proporciona la clase `Serializer` que combina serialización de datos con firma criptográfica, similar a la interfaz de `json`.

## Interfaz básica

La clase  `Serializer` envuelve un firmante para serializar y firmar datos que no son bytes. Proporciona métodos `dumps` y `loads` análogos a los del módulo `json` de Python.

Para serializar y firmar datos:

```python
from itsdangerous.serializer import Serializer
s = Serializer("secret-key")
s.dumps([1, 2, 3, 4])
# b'[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo'
```

Para deserializar y verificar la firma:

```python
s.loads('[1, 2, 3, 4].r7R9RhGgDPvvWl3iNzLuIIfELmo')
# [1, 2, 3, 4]
```

## Formatos soportados

 Por defecto, el serializador usa el módulo `json` de Python para serializar datos a una cadena, pero esto puede cambiarse.  El sistema es agnóstico al formato: los tests demuestran que funciona tanto con JSON como con `pickle`.

La clase  `_CompactJSON` es un envoltorio que genera JSON compacto sin espacios en blanco innecesarios, utilizando separadores `(",", ":")` para minimizar el tamaño.

## Integración con firma

El `Serializer`  serializa objetos a bytes mediante `dump_payload`, que siempre retorna bytes (codificando a UTF-8 si es necesario). Luego  `dumps` firma esa carga útil usando `make_signer` y retorna el resultado como bytes o texto según el tipo del serializador interno.

 El método `loads` verifica la firma descifrando el contenido con `iter_unsigners` e intentando deserializar la carga útil.

## Manejo de errores

Cuando la verificación de firma falla,  `loads_unsafe` permite acceder al contenido sin verificación (para depuración). Retorna una tupla `(signature_valid, payload)` donde el primer elemento es un booleano.

Para inspeccionar la carga útil en caso de error deliberadamente:

```python
try:
    decoded_payload = s.loads(data)
except BadSignature as e:
    if e.payload is not None:
        try:
            decoded_payload = s.load_payload(e.payload)
        except BadPayload:
            pass
        # La carga útil está decodificada pero no verificada
```

## Rotación de claves

 El `Serializer` permite pasar una lista de claves secretas para soportar rotación: las claves más antiguas se usan para verificar, y la más nueva se usa para firmar.

 Se puede configurar una lista de `fallback_signers` que se intentarán si la verificación con el firmante actual falla.  Esto permite, por ejemplo, cambiar el algoritmo de resumen (digest method) sin invalidar firmas antiguas.

## Persistencia en archivos

 El método `dump` escribe el resultado de `dumps` en un archivo.  El método `load` lee del archivo y llama a `loads`.

## Parámetros de configuración

 Al construir un `Serializer` se pueden especificar:
- `secret_key`: Clave secreta para firmar (puede ser una lista para rotación)
- `salt`: Valor adicional para distinguir contextos de firma
- `serializer`: Módulo de serialización personalizado (por defecto `json`)
- `serializer_kwargs`: Argumentos para pasar al método `dumps` del serializador
- `signer`: Clase de firmante personalizada
- `signer_kwargs`: Argumentos para la clase firmante
- `fallback_signers`: Lista de firmantes alternativos para verificación

Véase [Firma de datos](signer.md) para detalles sobre el sistema de firma subyacente.
