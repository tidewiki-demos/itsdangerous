# Guía de Marcas Temporales

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Las marcas temporales permiten añadir información de fecha y hora a los tokens firmados, de modo que se pueda verificar que un token no ha envejecido más allá de un límite aceptable. Esto es útil para garantizar que las credenciales, enlaces de verificación o datos sensibles sigan siendo válidos dentro de una ventana de tiempo específica.

## Concepto básico

Una marca temporal se incrusta en el token durante la firma y se incluye como parte de la firma criptográfica. Al verificar el token, el sistema comprueba no solo que la firma sea válida, sino también que la marca temporal esté dentro del rango de edad permitido. [`docs/timed.rst:6-9`](../../docs/timed.rst#L6-L9)

## Uso de TimestampSigner

La clase `TimestampSigner` es la herramienta principal para trabajar con marcas temporales. Permite firmar datos incluyendo automáticamente la hora actual, y luego validar que la firma no sea demasiado antigua. [`docs/timed.rst:13-15`](../../docs/timed.rst#L13-L15)

### Ejemplo básico

```python
from itsdangerous import TimestampSigner
s = TimestampSigner('secret-key')
string = s.sign('foo')
```

El método `sign()` añade la marca temporal actual al token y lo firma junto con los datos.

### Verificación con límite de edad

Al deserializar, se puede especificar la edad máxima permitida en segundos mediante el parámetro `max_age`:

```python
s.unsign(string, max_age=5)
```

Si la marca temporal es más antigua que el límite especificado, se lanza una excepción `SignatureExpired`. [`docs/timed.rst:19-22`](../../docs/timed.rst#L19-L22)

## Serialización con marca temporal

Para trabajar con estructuras de datos complejas (en lugar de solo cadenas), utiliza `TimedSerializer`, que combina la funcionalidad de [serialización](serialization-guide.md) con las marcas temporales. [`docs/timed.rst:27-28`](../../docs/timed.rst#L27-L28)

## Integración con la arquitectura

- Las marcas temporales se implementan como extensión de la [firma criptográfica](signing-guide.md) base.
- El tiempo se serializa como parte de los datos firmados, utilizando el sistema de [serialización](serializer.md) del repositorio.
- Las excepciones relacionadas con marcas temporales expiradas se documentan en [Validación con Marca Temporal](timestamp.md) y [Gestión de Errores](exceptions.md).
- Para tokens seguros para URL que incluyan marcas temporales, consulta [Guía de Tokens Seguros para URL](url-safe-guide.md).

## Casos de uso comunes

- **Enlaces de verificación por correo**: Generan un token que solo es válido durante un período limitado (ej: 24 horas).
- **Sesiones temporales**: Aseguran que una sesión no pueda ser reutilizada después de cierto tiempo.
- **Credenciales de corta vida**: Tokens que expiran automáticamente después de minutos u horas.
