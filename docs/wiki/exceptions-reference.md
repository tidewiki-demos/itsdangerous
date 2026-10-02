# Referencia de Excepciones

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

La librería define un conjunto de excepciones especializadas para diferentes tipos de errores que pueden ocurrir durante la firma criptográfica, validación de datos y procesamiento de marcas temporales. Todas las excepciones se encuentran en el módulo `itsdangerous.exc` [`docs/exceptions.rst:1`](../../docs/exceptions.rst#L1).

## Jerarquía de excepciones

Las excepciones de la librería forman una jerarquía que permite capturar errores en diferentes niveles de especificidad:

- **BadData**: excepción base para todos los errores relacionados con datos inválidos
  - **BadSignature**: firma criptográfica inválida o no verificada
    - **BadTimeSignature**: firma con marca temporal inválida
    - **SignatureExpired**: firma válida pero expirada
  - **BadHeader**: encabezado de datos malformado
  - **BadPayload**: carga de datos corrupta o inválida

## Excepciones principales

### BadData [`docs/exceptions.rst:6`](../../docs/exceptions.rst#L6)

Excepción base para todos los errores de validación de datos. Se lanza cuando los datos no cumplen con el formato o estructura esperados. Capturar esta excepción permite manejar cualquier error de validación de la librería de forma genérica.

### BadSignature [`docs/exceptions.rst:9`](../../docs/exceptions.rst#L9)

Se lanza cuando la firma criptográfica es inválida, no coincide con los datos o no puede ser verificada. Ocurre típicamente cuando:
- Los datos han sido modificados después de ser firmados
- Se utilizó una clave diferente para verificar
- La firma está corrupta

Consulta [Firmante: Base de Criptografía](core-signer.md) para entender cómo se generan y verifican firmas.

### BadTimeSignature [`docs/exceptions.rst:12`](../../docs/exceptions.rst#L12)

Variante especializada de `BadSignature` que se lanza específicamente cuando la firma que incluye una marca temporal es inválida. Consulta [Validación con Marca Temporal](timestamp.md) para más detalles sobre firmas con marca temporal.

### SignatureExpired [`docs/exceptions.rst:15`](../../docs/exceptions.rst#L15)

Se lanza cuando una firma es válida criptográficamente, pero su marca temporal ha expirado. Esta excepción permite diferenciar entre una firma inválida y una firma expirada. Consulta [Guía de Marcas Temporales](timestamp-guide.md) para configurar y validar tiempos de expiración.

### BadHeader [`docs/exceptions.rst:18`](../../docs/exceptions.rst#L18)

Se lanza cuando el encabezado que contiene metadatos de la firma está malformado, es incompleto o no puede ser parseado. Ocurre generalmente antes de intentar validar la firma misma.

### BadPayload [`docs/exceptions.rst:21`](../../docs/exceptions.rst#L21)

Se lanza cuando la carga de datos (payload) es corrupta, está mal comprimida, tiene una codificación inválida o no puede ser deserializada. Consulta [Codificación y Compresión de Datos](encoding.md) y [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) para entender el procesamiento de datos.

## Estrategias de manejo

### Captura genérica de errores de datos

```python
from itsdangerous import BadData

try:
    # Operación de firma o validación
    resultado = signer.unsign(datos)
except BadData as e:
    # Manejo genérico: cualquier error de validación de datos
    print(f"Datos inválidos: {e}")
```

### Captura diferenciada de errores de firma y expiración

```python
from itsdangerous import BadSignature, SignatureExpired

try:
    resultado = signer.unsign(datos)
except SignatureExpired:
    # La firma es válida pero expiró
    print("La firma ha expirado")
except BadSignature:
    # La firma es inválida o no verificable
    print("La firma es inválida")
```

### Captura completa de todos los errores

```python
from itsdangerous import (
    BadData, BadSignature, BadTimeSignature,
    SignatureExpired, BadHeader, BadPayload
)

try:
    resultado = signer.unsign(datos)
except SignatureExpired:
    print("Firma expirada")
except BadTimeSignature:
    print("Firma con marca temporal inválida")
except BadSignature:
    print("Firma inválida")
except BadHeader:
    print("Encabezado malformado")
except BadPayload:
    print("Carga de datos corrupta")
except BadData:
    print("Error de validación general")
```

## Relación con otros componentes

- Consulta [Gestión de Errores](exceptions.md) para patrones generales de manejo de errores
- Consulta [Guía de Firma Criptográfica](signing-guide.md) para ejemplos de uso con firmas
- Consulta [Referencia de API Pública](api-reference.md) para ver qué métodos lanzan cada excepción
