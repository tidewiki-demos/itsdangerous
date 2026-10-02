# Manejo de JSON

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Proporciona utilidades para serialización JSON con manejo especial de tipos de datos personalizados. El módulo facilita la conversión entre objetos Python y representaciones JSON de forma compacta, eliminando espacios innecesarios.

## Funcionalidad Principal

El módulo expone la clase `_CompactJSON` [`src/itsdangerous/_json.py:7-18`](../../src/itsdangerous/_json.py#L7-L18), que envuelve la funcionalidad estándar de `json` de Python con configuración optimizada para minimizar el tamaño del payload.

### Carga de JSON

El método `loads` [`src/itsdangerous/_json.py:11-12`](../../src/itsdangerous/_json.py#L11-L12) deserializa una cadena o bytes en un objeto Python. Acepta tanto strings como bytes, delegando directamente al módulo `json` estándar.

### Serialización Compacta

El método `dumps` [`src/itsdangerous/_json.py:15-17`](../../src/itsdangerous/_json.py#L15-L17) convierte objetos Python a JSON con dos optimizaciones:

- `ensure_ascii=False`: Permite caracteres Unicode nativos en lugar de escaparlos, reduciendo la longitud del resultado
- `separators=(",", ":")`: Utiliza separadores mínimos sin espacios entre elementos, eliminando whitespace innecesario

Ambas configuraciones se aplican por defecto, pero pueden sobrescribirse pasando argumentos adicionales en `**kwargs`.

## Casos de Uso

Este módulo se utiliza internamente en el sistema de serialización para garantizar que los datos JSON serializado sean lo más compactos posible, lo cual es especialmente importante cuando se integra con:

- [Serializador: Almacenamiento de Estructuras de Datos](serializer.md) — manejo de estructuras complejas
- [Firmante: Base de Criptografía](core-signer.md) — donde el tamaño del payload afecta el resultado criptográfico
- [Codificación Segura para URL](url-safe.md) — minimizar longitud mejora la codificación URL
