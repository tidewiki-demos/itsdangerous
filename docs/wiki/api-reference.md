# Referencia de API Pública

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Define la interfaz pública del paquete itsdangerous, exponiendo las clases y funciones principales para uso de desarrolladores externos. Todos los símbolos listados aquí están disponibles importando directamente desde el paquete raíz.

## Clases Principales

### Firmado y Serialización

**`Signer`** [`src/itsdangerous/__init__.py:17`](../../src/itsdangerous/__init__.py#L17)
Clase base para firmar datos. Proporciona los métodos fundamentales de firma criptográfica. Consulta [Firmante: Base de Criptografía](core-signer.md) y [Guía de Firma Criptográfica](signing-guide.md) para detalles de uso.

**`Serializer`** [`src/itsdangerous/__init__.py:14`](../../src/itsdangerous/__init__.py#L14)
Combina serialización y firma de datos. Permite convertir estructuras de datos a formato firmado y verificarlas. Ver [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) y [Guía de Serialización](serialization-guide.md).

**`TimestampSigner`** [`src/itsdangerous/__init__.py:19`](../../src/itsdangerous/__init__.py#L19)
Extiende `Signer` agregando marca temporal a las firmas. Útil para crear firmas que expiren después de un tiempo determinado. Consulta [Validación con Marca Temporal](timestamp.md) y [Guía de Marcas Temporales](timestamp-guide.md).

**`TimedSerializer`** [`src/itsdangerous/__init__.py:18`](../../src/itsdangerous/__init__.py#L18)
Combina serialización con soporte de marca temporal. Permite firmar datos que se validan según antigüedad. Ver [Validación con Marca Temporal](timestamp.md).

**`URLSafeSerializer`** [`src/itsdangerous/__init__.py:20`](../../src/itsdangerous/__init__.py#L20)
Serializa datos en formato seguro para usar en URLs. Utiliza codificación que evita caracteres problemáticos en URLs. Consulta [Codificación Segura para URL](url-safe.md) y [Guía de Tokens Seguros para URL](url-safe-guide.md).

**`URLSafeTimedSerializer`** [`src/itsdangerous/__init__.py:21`](../../src/itsdangerous/__init__.py#L21)
Combina serialización segura para URL con soporte de marca temporal. Ideal para tokens de confirmación por correo con expiración.

## Algoritmos de Firma

**`HMACAlgorithm`** [`src/itsdangerous/__init__.py:15`](../../src/itsdangerous/__init__.py#L15)
Algoritmo HMAC para firmar datos. Es el algoritmo predeterminado y se usa internamente en `Signer` y derivados. Consulta [Firmante: Base de Criptografía](core-signer.md).

**`NoneAlgorithm`** [`src/itsdangerous/__init__.py:16`](../../src/itsdangerous/__init__.py#L16)
Algoritmo que no realiza firma real (solo añade un separador). Utilizado principalmente para verificar comportamientos o en contextos donde la firma no es crítica.

## Funciones de Codificación

**`base64_encode()`** [`src/itsdangerous/__init__.py:6`](../../src/itsdangerous/__init__.py#L6)
Codifica bytes a string base64. Consulta [Codificación y Compresión de Datos](encoding.md) y [Guía de Codificación](encoding-guide.md).

**`base64_decode()`** [`src/itsdangerous/__init__.py:5`](../../src/itsdangerous/__init__.py#L5)
Decodifica string base64 a bytes.

**`want_bytes()`** [`src/itsdangerous/__init__.py:7`](../../src/itsdangerous/__init__.py#L7)
Convierte un valor (string o bytes) a bytes, asegurando representación uniforme para procesamiento interno.

## Excepciones

Todas las excepciones heredan de una jerarquía común. Ver [Gestión de Errores](exceptions.md) y [Referencia de Excepciones](exceptions-reference.md) para el manejo detallado.

**`BadData`** [`src/itsdangerous/__init__.py:8`](../../src/itsdangerous/__init__.py#L8)
Excepción base para datos inválidos o no confiables.

**`BadSignature`** [`src/itsdangerous/__init__.py:11`](../../src/itsdangerous/__init__.py#L11)
Se lanza cuando la firma no es válida o no coincide.

**`BadHeader`** [`src/itsdangerous/__init__.py:9`](../../src/itsdangerous/__init__.py#L9)
Se lanza cuando la cabecera del dato firmado es inválida.

**`BadPayload`** [`src/itsdangerous/__init__.py:10`](../../src/itsdangerous/__init__.py#L10)
Se lanza cuando la carga útil (payload) no puede ser deserializada correctamente.

**`BadTimeSignature`** [`src/itsdangerous/__init__.py:12`](../../src/itsdangerous/__init__.py#L12)
Se lanza cuando hay problema con la firma de marca temporal.

**`SignatureExpired`** [`src/itsdangerous/__init__.py:13`](../../src/itsdangerous/__init__.py#L13)
Se lanza cuando una firma con marca temporal ha expirado.

## Notas de Importación

Todos los símbolos enumerados se importan explícitamente en el módulo `__init__.py` [`src/itsdangerous/__init__.py:1-21`](../../src/itsdangerous/__init__.py#L1-L21), haciendo que estén disponibles directamente:

```python
from itsdangerous import Signer, Serializer, TimestampSigner, URLSafeSerializer
from itsdangerous import BadSignature, SignatureExpired
```

No se debe acceder a atributos internos o símbolos no documentados aquí, ya que su compatibilidad entre versiones no está garantizada.
