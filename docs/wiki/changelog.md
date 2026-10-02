# Registro de Cambios

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Documentación de todas las características, correcciones de errores y cambios importantes en cada versión de itsdangerous.

## Versión 2.2.0

**Lanzamiento:** 2024-04-16

- Eliminado soporte para Python 3.7
- Empaquetado modernizado con `pyproject.toml` en lugar de `setup.cfg`, usando `flit_core` como backend en lugar de `setuptools`
- Atributo `__version__` deprecado. Se recomienda usar detección de características o `importlib.metadata.version("itsdangerous")`
- `Serializer` y el tipo de retorno de `dumps` son genéricos para verificación de tipos. Por defecto es `Serializer[str]` y `dumps` retorna `str`. Con un argumento `serializer` diferente, intenta inferir el tipo de retorno de su método `dumps`
- `hashlib.sha1` por defecto ya no se accede en tiempo de importación para permitir que desarrolladores cambien el default en compilaciones FIPS

## Versión 2.1.2

**Lanzamiento:** 2022-03-24

- Manejo de desbordamiento de fechas en desifra con marca temporal en sistemas de 32-bit

## Versión 2.1.1

**Lanzamiento:** 2022-03-09

- Manejo de desbordamiento de fechas en desifra con marca temporal

## Versión 2.1.0

**Lanzamiento:** 2022-02-17

- Eliminado soporte para Python 3.6
- Eliminación de código previamente deprecado:
  - Funcionalidad JWS: usar bibliotecas dedicadas como Authlib
  - `import itsdangerous.json`: importar `json` de la biblioteca estándar

## Versión 2.0.1

**Lanzamiento:** 2021-05-18

- Nombres de nivel superior marcados como exportados para que verificadores de tipos entiendan importaciones en proyectos de usuarios
- El argumento `salt` a `Serializer` y `Signer` puede ser `None` nuevamente

## Versión 2.0.0

**Lanzamiento:** 2021-05-11

- Eliminado soporte para Python 2 y 3.5
- Soporte JWS (`JSONWebSignatureSerializer`, `TimedJSONWebSignatureSerializer`) deprecado. Se recomienda usar bibliotecas dedicadas JWS/JWT como authlib
- Importar `itsdangerous.json` deprecado. Importar el módulo `json` de Python en su lugar
- Simplejson ya no se usa si está instalado. Para usar una biblioteca diferente, pasarla como `Serializer(serializer=...)`
- Valores `datetime` son conscientes de zona horaria con `timezone.utc`. Código usando `TimestampSigner.unsign(return_timestamp=True)` o `BadTimeSignature.date_signed` puede necesitar cambios
- Si una firma tiene una edad menor a 0, lanza `SignatureExpired` en lugar de aparecer válida
- `BadTimeSignature.date_signed` siempre es un objeto `datetime` en lugar de `int` en algunos casos
- Agregado soporte para rotación de claves. Una lista de claves puede pasarse como `secret_key`, de más antigua a más nueva. La clave más nueva se usa para firmar, todas las claves se intenta para desifrar
- Eliminado el firmante fallback SHA-512 por defecto de `default_fallback_signers`
- Información de tipos agregada para herramientas de tipado estático

## Versión 1.1.0

**Lanzamiento:** 2018-10-26

- Algoritmo de firma por defecto cambiado nuevamente a SHA-1
- Agregado fallback SHA-512 por defecto para usuarios del lanzamiento 1.0.0 (retirado) que usaba SHA-512
- Soporte para algoritmos fallback durante deserialización para permitir cambiar el default en el futuro sin romper firmas existentes
- Capitalización de paquetes cambiada nuevamente a minúsculas

## Versión 1.0.0

**Lanzamiento:** 2018-10-18

**RETIRADO**

*Nota:* Este lanzamiento fue retirado de PyPI porque cambió el algoritmo por defecto a SHA-512. Esta decisión fue revertida en 1.1.0 y permanece en SHA1.

- Eliminado soporte para Python 2.6 y 3.3
- Refactorización de código de módulo único a paquete. Cualquier objeto en la documentación API aún es importable desde el nombre `itsdangerous` de nivel superior, pero otras importaciones necesitarán cambios
- Optimización de serialización y deserialización de marcas temporales
- `base64_decode` lanza `BadData` cuando se pasa datos inválidos
- Asegurado que valor sea bytes al firmar para evitar `TypeError` en Python 3
- Argumento `serializer_kwargs` agregado a `Serializer`, pasado a `dumps` durante `dump_payload`
- Volcados JSON más compactos para strings unicode
- Marca temporal completa utilizada en lugar de offset, permitiendo fechas antes de 2011
- Detecta carácter `sep` que puede aparecer en la firma misma y lanza `ValueError`
- Firma consistente para argumentos con palabra clave en `Serializer.load_payload` en subclases
- Hash intermedio por defecto cambiado de SHA-1 a SHA-512
- Encabezado exp de JWS convertido a int al cargar

## Versión 0.24

**Lanzamiento:** 2014-03-28

- Excepción `BadHeader` agregada para encabezados inválidos, reemplazando la antigua excepción `BadPayload` reutilizada en esos casos

## Versión 0.23

**Lanzamiento:** 2013-08-08

- Error de empaquetamiento corregido que causaba que archivos de pruebas y licencia no se incluyeran

## Versión 0.22

**Lanzamiento:** 2013-07-03

- Soporte para `TimedJSONWebSignatureSerializer` agregado
- Posibilidad de anular la función de verificación de firma para permitir implementar algoritmos asimétricos

## Versión 0.21

**Lanzamiento:** 2013-05-26

- Problema en Python 3 que causaba generación de errores inválidos corregido

## Versión 0.20

**Lanzamiento:** 2013-05-23

- Llamada incorrecta a `want_bytes` que rompía algunos usos de ItsDangerous en Python 2.6 corregida

## Versión 0.19

**Lanzamiento:** 2013-05-21

- Soporte para Python 2.5 eliminado y soporte para 3.3 agregado

## Versión 0.18

**Lanzamiento:** 2013-05-03

- Soporte para JSON Web Signatures (JWS) agregado

## Versión 0.17

**Lanzamiento:** 2012-08-10

- Error de nombre al anular el método digest corregido

## Versión 0.16

**Lanzamiento:** 2012-07-11

- Posibilidad de pasar valores unicode a `load_payload` para facilitar depuración

## Versión 0.15

**Lanzamiento:** 2012-07-11

- `load_payload` independiente más robusta con error específico si algo falla
- Excepciones refactorizadas para capturar más casos individualmente, atributos adicionales
- Problema que causaba que `load_payload` no funcionara en algunas situaciones con serializadores basados en marca temporal corregido
- Método `loads_unsafe` agregado

## Versión 0.14

**Lanzamiento:** 2012-06-29

- Refactorización de API para soportar diferentes derivaciones de claves
- Atributos agregados a excepciones para inspeccionar datos incluso si la verificación de firma falló

## Versión 0.13

**Lanzamiento:** 2012-06-10

- Pequeño cambio de API que permite personalización del módulo digest

## Versión 0.12

**Lanzamiento:** 2012-02-22

- Problema con zona horaria local usada para cálculo de epoch corregido. Esto puede invalidar algunas de sus firmas si no estaba ejecutando en zona horaria UTC. Es posible revertir al comportamiento antiguo haciendo monkey patching a `itsdangerous.EPOCH`

## Versión 0.11

**Lanzamiento:** 2011-07-07

- Error de valor no capturado corregido

## Versión 0.10

**Lanzamiento:** 2011-06-25

- Interfaz refactorizada permitiendo intercambiar serializadores subyacentes pasando un módulo en lugar de anular cargadores y volcadores de payload. Esto hace la interfaz más compatible con cambios recientes de Django
