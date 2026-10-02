# Guía de Tokens Seguros para URL

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Esta guía explica cómo generar y validar tokens que pueden usarse de forma segura en URLs y parámetros de consulta sin caracteres problemáticos.

## Visión General

Los tokens seguros para URL son cadenas serializadas y firmadas criptográficamente que solo contienen caracteres seguros para usar en URLs (alfanuméricos, guiones y guiones bajos). Esto permite pasar datos de confianza en URLs sin preocuparse por caracteres especiales que requieran codificación adicional.

## Serialización Segura para URL

[`docs/url_safe.rst:12-17`](../../docs/url_safe.rst#L12-L17) La forma más básica de generar un token seguro para URL es usar `URLSafeSerializer`:

```python
from itsdangerous.url_safe import URLSafeSerializer

s = URLSafeSerializer("secret-key")
token = s.dumps([1, 2, 3, 4])
# resultado: 'WzEsMiwzLDRd.wSPHqC0gR7VUqivlSukJ0IeTDgo'

datos = s.loads(token)
# resultado: [1, 2, 3, 4]
```

El token generado contiene solo caracteres seguros para URL y puede pasarse directamente como parámetro de consulta sin necesidad de codificación URL adicional.

## Con Marcas Temporales

[`docs/url_safe.rst:21`](../../docs/url_safe.rst#L21) Para tokens que expiran después de un tiempo específico, usa `URLSafeTimedSerializer`. Esto es útil para tokens de confirmación de correo, restablecimiento de contraseña y otras operaciones sensibles al tiempo:

```python
from itsdangerous.url_safe import URLSafeTimedSerializer

s = URLSafeTimedSerializer("secret-key")
token = s.dumps({"email": "user@example.com"})

# Validar el token dentro de un periodo específico (ej: 3600 segundos)
datos = s.loads(token, max_age=3600)
```

Consulta [Validación con Marca Temporal](timestamp.md) para más detalles sobre el manejo de tiempos de expiración.

## Casos de Uso Comunes

- **Tokens de Confirmación de Email**: Genera un token con la dirección de correo, envíalo en un enlace de confirmación y valida que sea válido dentro de 24 horas
- **Restablecimiento de Contraseña**: Crea un token con el ID de usuario que expira en 1 hora
- **Tokens de Sesión Temporal**: Serializa datos de sesión que pueden transmitirse de forma segura en URLs

## Diferencia con Serialización Regular

La diferencia clave entre `URLSafeSerializer` y [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) es la codificación:

- `URLSafeSerializer`: Usa codificación base64 URL-safe (caracteres `A-Za-z0-9-_`)
- `Serializer` regular: Puede contener caracteres que requieren codificación URL

Ambos utilizan la misma [Firmante: Base de Criptografía](core-signer.md) para garantizar la integridad.

## Conceptos Relacionados

- [Codificación Segura para URL](url-safe.md): Detalles técnicos de la codificación
- [Guía de Firma Criptográfica](signing-guide.md): Cómo funcionan las firmas
- [Guía de Serialización](serialization-guide.md): Serialización de datos en general
- [Guía de Marcas Temporales](timestamp-guide.md): Expiración de tokens basada en tiempo
