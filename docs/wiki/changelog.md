# Historial de cambios

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Registro de versiones del proyecto itsdangerous con descripción de cambios, mejoras, correcciones y decisiones importantes de cada lanzamiento.

## Versión 2.2.0

**Lanzada:** 16 de abril de 2024

- Se descartó soporte para Python 3.7 [`CHANGES.rst:6`](../../CHANGES.rst#L6)
- Migración a metadatos modernos con `pyproject.toml` en lugar de `setup.cfg` [`CHANGES.rst:7-8`](../../CHANGES.rst#L7-L8)
- Cambio de backend de construcción a `flit_core` en lugar de `setuptools` [`CHANGES.rst:9`](../../CHANGES.rst#L9)
- Deprecación del atributo `__version__`. Se recomienda usar detección de características o `importlib.metadata.version("itsdangerous")` [`CHANGES.rst:10-11`](../../CHANGES.rst#L10-L11)
- `Serializer` y el tipo de retorno de `dumps` ahora son genéricos para verificación de tipos. Por defecto es `Serializer[str]` devolviendo `str`. Con un argumento `serializer` diferente, se intenta inferir el tipo de retorno de su método `dumps` [`CHANGES.rst:12-15`](../../CHANGES.rst#L12-L15) (ver [Serialización y compresión](serializer.md))
- Mejora en manejo de `hashlib.sha1` en compilaciones FIPS: no se accede en tiempo de importación para permitir cambiar el valor por defecto [`CHANGES.rst:16-18`](../../CHANGES.rst#L16-L18)

## Versión 2.1.2

**Lanzada:** 24 de marzo de 2022

- Manejo de desbordamiento de fecha en unsign temporizado en sistemas de 32 bits [`CHANGES.rst:26`](../../CHANGES.rst#L26)

## Versión 2.1.1

**Lanzada:** 9 de marzo de 2022

- Manejo de desbordamiento de fecha en unsign temporizado [`CHANGES.rst:34`](../../CHANGES.rst#L34) (ver [Verificación con marca de tiempo](timestamp-verification.md))

## Versión 2.1.0

**Lanzada:** 17 de febrero de 2022

- Se descartó soporte para Python 3.6 [`CHANGES.rst:42`](../../CHANGES.rst#L42)
- Remoción de código previamente deprecado [`CHANGES.rst:43-48`](../../CHANGES.rst#L43-L48):
  - Funcionalidad JWS: se recomienda usar librerías dedicadas como Authlib
  - `import itsdangerous.json`: importar `json` de la librería estándar

## Versión 2.0.1

**Lanzada:** 18 de mayo de 2021

- Marcado de nombres de nivel superior como exportados para que la verificación de tipos entienda importaciones en proyectos de usuarios [`CHANGES.rst:56-57`](../../CHANGES.rst#L56-L57)
- El argumento `salt` en `Serializer` y `Signer` puede ser `None` nuevamente [`CHANGES.rst:58-59`](../../CHANGES.rst#L58-L59)

## Versión 2.0.0

**Lanzada:** 11 de mayo de 2021

- Se descartó soporte para Python 2 y 3.5 [`CHANGES.rst:67`](../../CHANGES.rst#L67)
- Deprecación de soporte JWS (`JSONWebSignatureSerializer`, `TimedJSONWebSignatureSerializer`). Se recomienda usar librerías dedicadas como authlib [`CHANGES.rst:68-70`](../../CHANGES.rst#L68-L70)
- Importar `itsdangerous.json` está deprecado. Usar módulo `json` de Python [`CHANGES.rst:71-72`](../../CHANGES.rst#L71-L72)
- Simplejson ya no se usa si está instalado. Para usar otra librería, pasarla con `Serializer(serializer=...)` [`CHANGES.rst:73-74`](../../CHANGES.rst#L73-L74)
- Valores `datetime` son conscientes de zona horaria con `timezone.utc`. Código usando `TimestampSigner.unsign(return_timestamp=True)` o `BadTimeSignature.date_signed` puede necesitar cambios [`CHANGES.rst:75-77`](../../CHANGES.rst#L75-L77) (ver [Verificación con marca de tiempo](timestamp-verification.md))
- Si una firma tiene antigüedad menor a 0, lanza `SignatureExpired` en lugar de aparecer válida (puede ocurrir si se cambia el offset de marca de tiempo) [`CHANGES.rst:78-80`](../../CHANGES.rst#L78-L80)
- `BadTimeSignature.date_signed` siempre es objeto `datetime` en lugar de `int` en algunos casos [`CHANGES.rst:81-82`](../../CHANGES.rst#L81-L82)
- Soporte agregado para rotación de claves. Una lista de claves puede pasarse como `secret_key`, de más antigua a más nueva. La clave más nueva se usa para firmar; todas se prueban para unsign [`CHANGES.rst:83-85`](../../CHANGES.rst#L83-L85)
- Removido el firmante por defecto de respaldo SHA-512 de `default_fallback_signers` [`CHANGES.rst:86-87`](../../CHANGES.rst#L86-L87)
- Se agregó información de tipos para herramientas de escritura estática [`CHANGES.rst:88`](../../CHANGES.rst#L88)

## Versión 1.1.0

**Lanzada:** 26 de octubre de 2018

- Cambio del algoritmo de firma por defecto nuevamente a SHA-1 [`CHANGES.rst:96`](../../CHANGES.rst#L96)
- SHA-512 por defecto agregado como respaldo para usuarios que usaron el lanzamiento retirado 1.0.0 que defaulteaba a SHA-512 [`CHANGES.rst:97-98`](../../CHANGES.rst#L97-L98)
- Soporte agregado para algoritmos de respaldo durante deserialización para permitir cambiar el valor por defecto en el futuro sin romper firmas existentes [`CHANGES.rst:99-101`](../../CHANGES.rst#L99-L101)
- Capitalización de paquetes cambiada nuevamente a minúsculas; el cambio anterior rompió alguna herramienta [`CHANGES.rst:102-103`](../../CHANGES.rst#L102-L103)

## Versión 1.0.0

**Lanzada:** 18 de octubre de 2018

**RETIRADO**

Este lanzamiento fue retirado de PyPI porque cambió el algoritmo por defecto a SHA-512. Esta decisión fue revertida en 1.1.0 y permanece en SHA-1 [`CHANGES.rst:111-115`](../../CHANGES.rst#L111-L115).

- Se descartó soporte para Python 2.6 y 3.3 [`CHANGES.rst:117`](../../CHANGES.rst#L117)
- Refactorización de código de módulo único a paquete. Cualquier objeto en la documentación de API sigue importable desde el nombre `itsdangerous` de nivel superior, pero otras importaciones necesitarán cambios. Futuras versiones removerán muchas importaciones de compatibilidad [`CHANGES.rst:118-121`](../../CHANGES.rst#L118-L121)
- Optimización de cómo se serializan y deserializan marcas de tiempo [`CHANGES.rst:122`](../../CHANGES.rst#L122)
- `base64_decode` lanza `BadData` cuando recibe datos inválidos [`CHANGES.rst:123-124`](../../CHANGES.rst#L123-L124)
- Asegurar que el valor es bytes al firmar para evitar `TypeError` en Python 3 [`CHANGES.rst:125-126`](../../CHANGES.rst#L125-L126)
- Argumento `serializer_kwargs` agregado a `Serializer`, pasado a `dumps` durante `dump_payload` [`CHANGES.rst:127-128`](../../CHANGES.rst#L127-L128)
- Dumps JSON más compacto para strings unicode [`CHANGES.rst:129`](../../CHANGES.rst#L129)
- Usar marca de tiempo completa en lugar de offset, permitiendo fechas anteriores a 2011 [`CHANGES.rst:130-135`](../../CHANGES.rst#L130-L135)
- Detectar carácter `sep` que pueda aparecer en la firma misma y lanzar `ValueError` [`CHANGES.rst:136-137`](../../CHANGES.rst#L136-L137)
- Firma consistente para argumentos con palabras clave en `Serializer.load_payload` en subclases [`CHANGES.rst:138-139`](../../CHANGES.rst#L138-L139)
- Hash intermedio por defecto cambiado de SHA-1 a SHA-512 [`CHANGES.rst:140`](../../CHANGES.rst#L140)
- Conversión de header exp JWS a int al cargar [`CHANGES.rst:141`](../../CHANGES.rst#L141)

## Versión 0.24

**Lanzada:** 28 de marzo de 2014

- Excepción `BadHeader` agregada para encabezados malformados, reemplazando excepción `BadPayload` reutilizada [`CHANGES.rst:149-151`](../../CHANGES.rst#L149-L151)

## Versión 0.23

**Lanzada:** 8 de agosto de 2013

- Corrección de error de empaquetado que causaba que tests y archivos de licencia no se incluyeran [`CHANGES.rst:159-160`](../../CHANGES.rst#L159-L160)

## Versión 0.22

**Lanzada:** 3 de julio de 2013

- Soporte para `TimedJSONWebSignatureSerializer` agregado [`CHANGES.rst:168`](../../CHANGES.rst#L168)
- Posibilidad de sobreescribir función de verificación de firma para permitir implementar algoritmos asimétricos [`CHANGES.rst:169-170`](../../CHANGES.rst#L169-L170)

## Versión 0.21

**Lanzada:** 26 de mayo de 2013

- Corrección de problema en Python 3 que causaba errores inválidos [`CHANGES.rst:178-179`](../../CHANGES.rst#L178-L179)

## Versión 0.20

**Lanzada:** 23 de mayo de 2013

- Corrección de llamada incorrecta a `want_bytes` que rompía algunos usos de ItsDangerous en Python 2.6 [`CHANGES.rst:187-188`](../../CHANGES.rst#L187-L188)

## Versión 0.19

**Lanzada:** 21 de mayo de 2013

- Soporte descartado para 2.5, soporte agregado para 3.3 [`CHANGES.rst:196`](../../CHANGES.rst#L196)

## Versión 0.18

**Lanzada:** 3 de mayo de 2013

- Soporte para JSON Web Signatures (JWS) agregado [`CHANGES.rst:204`](../../CHANGES.rst#L204)

## Versión 0.17

**Lanzada:** 10 de agosto de 2012

- Corrección de error de nombre al sobreescribir método de digest [`CHANGES.rst:212`](../../CHANGES.rst#L212)

## Versión 0.16

**Lanzada:** 11 de julio de 2012

- Posibilidad de pasar valores unicode a `load_payload` para facilitar depuración [`CHANGES.rst:220-221`](../../CHANGES.rst#L220-L221)

## Versión 0.15

**Lanzada:** 11 de julio de 2012

- `load_payload` independiente más robusto lanzando error específico si algo falla [`CHANGES.rst:229-230`](../../CHANGES.rst#L229-L230)
- Refactorización de excepciones para capturar más casos individualmente, atributos agregados [`CHANGES.rst:231-232`](../../CHANGES.rst#L231-L232)
- Corrección de problema que causaba que `load_payload` no funcionara en algunas situaciones con serializadores basados en marca de tiempo [`CHANGES.rst:233-234`](../../CHANGES.rst#L233-L234)
- Método `loads_unsafe` agregado [`CHANGES.rst:235`](../../CHANGES.rst#L235)

## Versión 0.14

**Lanzada:** 29 de junio de 2012

- Refactorización de API para soportar diferentes derivaciones de clave [`CHANGES.rst:243`](../../CHANGES.rst#L243)
- Atributos agregados a excepciones para inspeccionar datos aunque falle verificación de firma [`CHANGES.rst:244-245`](../../CHANGES.rst#L244-L245)

## Versión 0.13

**Lanzada:** 10 de junio de 2012

- Pequeño cambio de API que permite personalización del módulo digest [`CHANGES.rst:253`](../../CHANGES.rst#L253)

## Versión 0.12

**Lanzada:** 22 de febrero de 2012

- Corrección de problema con zona horaria local usada en cálculo de época. Puede invalidar algunas firmas si no se ejecutaba en zona horaria UTC. Revertir a comportamiento antiguo con monkey patching `itsdangerous.EPOCH` [`CHANGES.rst:261-264`](../../CHANGES.rst#L261-L264)

## Versión 0.11

**Lanzada:** 7 de julio de 2011

- Corrección de error de valor no capturado [`CHANGES.rst:272`](../../CHANGES.rst#L272)

## Versión 0.10

**Lanzada:** 25 de junio de 2011

- Refactorización de interfaz para permitir intercambiar serializadores subyacentes pasando módulo en lugar de sobreescribir cargadores y descargadores de payload. Interfaz más compatible con cambios recientes de Django [`CHANGES.rst:280-283`](../../CHANGES.rst#L280-L283)
