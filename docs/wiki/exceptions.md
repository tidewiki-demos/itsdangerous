# Gestión de Errores

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Las excepciones personalizadas de ItsDangerous definen todos los errores que pueden ocurrir durante la firma, verificación y deserialización de datos. Forman una jerarquía que permite capturar errores específicos o genéricos según sea necesario.

## Jerarquía de Excepciones

[`src/itsdangerous/exc.py:7-19`](../../src/itsdangerous/exc.py#L7-L19)

`BadData` es la excepción base de la que derivan todas las demás. Almacena un mensaje descriptivo y lo expone a través de `__str__()`.

### Excepciones de Firma

[`src/itsdangerous/exc.py:22-33`](../../src/itsdangerous/exc.py#L22-L33)

`BadSignature` se lanza cuando una firma no coincide con los datos. Además del mensaje, puede incluir el payload que falló la verificación en el atributo `payload`, lo que permite inspeccionar los datos incluso si fueron alterados.

[`src/itsdangerous/exc.py:36-57`](../../src/itsdangerous/exc.py#L36-L57)

`BadTimeSignature` extiende `BadSignature` para firmas basadas en tiempo. Incluye el atributo `date_signed` con la fecha en que se generó la firma (timezone-aware). Esto es útil para informar al usuario cuánto tiempo lleva un enlace desactualizado.

[`src/itsdangerous/exc.py:60-63`](../../src/itsdangerous/exc.py#L60-L63)

`SignatureExpired` hereda de `BadTimeSignature` y se lanza cuando un timestamp de firma es más antiguo que el `max_age` permitido.

### Excepciones de Encabezado

[`src/itsdangerous/exc.py:66-89`](../../src/itsdangerous/exc.py#L66-L89)

`BadHeader` se lanza cuando un encabezado firmado es inválido. Ocurre solo en serializadores que incluyen un encabezado junto con la firma. Puede incluir:

- `header`: El encabezado si está disponible pero malformado.
- `original_error`: La excepción subyacente que causó que el encabezado fuese inválido.

### Excepciones de Payload

[`src/itsdangerous/exc.py:92-106`](../../src/itsdangerous/exc.py#L92-L106)

`BadPayload` se lanza cuando un payload es inválido. Esto puede ocurrir si:

- El payload se carga a pesar de una firma inválida.
- Hay una desconexión entre el serializador y deserializador.

El atributo `original_error` captura la excepción original que ocurrió durante la deserialización.

## Relación con Otros Componentes

- Las excepciones de firma se lanzan desde [Firmante: Base de Criptografía](core-signer.md) durante operaciones de firma y verificación.
- Las excepciones de payload ocurren en [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) durante deserialización.
- Las excepciones de marca temporal se lanzan desde [Validación con Marca Temporal](timestamp.md) cuando se validan datos con tiempo.

Para más detalles sobre cada excepción, consulta [Referencia de Excepciones](exceptions-reference.md).
