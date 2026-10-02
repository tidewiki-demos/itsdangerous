# Historial de cambios

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Registro de versiones del proyecto itsdangerous con descripción de cambios, mejoras, correcciones y decisiones importantes de cada lanzamiento.

## Versión 2.3.0

**Lanzada:** Sin lanzamiento (sin liberar)

- Se descartó soporte para Python 3.8 y 3.9 [`CHANGES.rst:6`](../../CHANGES.rst#L6)
- Remoción de código previamente deprecado [`CHANGES.rst:7`](../../CHANGES.rst#L7)

## Versión 2.2.0

**Lanzada:** 16 de abril de 2024

- Se descartó soporte para Python 3.7 [`CHANGES.rst:15`](../../CHANGES.rst#L15)
- Migración a metadatos modernos con `pyproject.toml` en lugar de `setup.cfg` [`CHANGES.rst:16-17`](../../CHANGES.rst#L16-L17)
- Cambio de backend de construcción a `flit_core` en lugar de `setuptools` [`CHANGES.rst:18`](../../CHANGES.rst#L18)
- Deprecación del atributo `__version__`. Se recomienda usar detección de características o `importlib.metadata.version("itsdangerous")` [`CHANGES.rst:19-20`](../../CHANGES.rst#L19-L20)
- `Serializer` y el tipo de retorno de `dumps` ahora son genéricos para verificación de tipos. Por defecto es `Serializer[str]` devolviendo `str`. Con un argumento `serializer` diferente, se intenta inferir el tipo de retorno de su método `dumps` [`CHANGES.rst:21-24`](../../CHANGES.rst#L21-L24) (ver [Serialización y compresión](serializer.md))
- Mejora en manejo de `hashlib.sha1` en compilaciones FIPS: no se accede en tiempo de importación para permitir cambiar el valor por defecto [`CHANGES.rst:25-27`](../../CHANGES.rst#L25-L27)

## Versión 2.1.2

**Lanzada:** 24 de marzo de 2022

- Manejo de desbordamiento de fecha en unsign temporizado en sistemas de 32 bits [`CHANGES.rst:35`](../../CHANGES.rst#L35)

## Versión 2.1.1

**Lanzada:** 9 de marzo de 2022

- Manejo de desbordamiento de fecha en unsign temporizado [`CHANGES.rst:43`](../../CHANGES.rst#L43) (ver [Verificación con marca de tiempo](timestamp-verification.md))

## Versión 2.1.0

**Lanzada:** 17 de febrero de 2022

- Se descartó soporte para Python 3.6 [`CHANGES.rst:51`](../../CHANGES.rst#L51)
- Remoción de código previamente deprecado [`CHANGES.rst:52-57`](../../CHANGES.rst#L52-L57):
  - Funcionalidad JWS: se recomienda usar librerías dedicadas como Authlib
  - `import itsdangerous.json`: importar `json` de la librería estándar

## Versión 2.0.1

**Lanzada:** 18 de mayo de 2021

- Marcado de nombres de nivel superior como exportados para que la verificación de tipos entienda importaciones en proyectos de usuarios [`CHANGES.rst:65-66`](../../CHANGES.rst#L65-L66)
- El argumento `salt` en `Serializer` y `Signer` puede ser `None` nuevamente [`CHANGES.rst:67-68`](../../CHANGES.rst#L67-L68)

## Versión 2.0.0

**Lanzada:** 11 de mayo de 2021

- Se descartó soporte para Python 2 y 3.5 [`CHANGES.rst:76`](../../CHANGES.rst#L76)
- Deprecación de soporte JWS (`JSONWebSignatureSerializer`, `TimedJSONWebSignatureSerializer`). Se recomienda usar librerías dedicadas como authlib [`CHANGES.rst:77-79`](../../CHANGES.rst#L77-L79)
- Importar `itsdangerous.json` está deprecado. Usar módulo `json` de Python [`CHANGES.rst:80-81`](../../CHANGES.rst#L80-L81)
- Simplejson ya no se usa si está instalado. Para usar otra librería, pasarla con `Serializer(serializer=...)` [`CHANGES.rst:82-83`](../../CHANGES.rst#L82-L83)
- Valores `datetime` son conscientes de zona horaria con `timezone.utc`. Código usando `TimestampSigner.unsign(return_timestamp=True)` o `BadTimeSignature.date_signed` puede necesitar cambios [`CHANGES.rst:84-86`](../../CHANGES.rst#L84-L86) (ver [Verificación con marca de tiempo](timestamp-verification.md))
- Si una firma tiene antigüedad menor a 0, lanza `SignatureExpired` en lugar de aparecer válida (puede ocurrir si se cambia el offset de marca de tiempo) [`CHANGES.rst:87-89`](../../CHANGES.rst#L87-L89)
- `BadTimeSignature.date_signed` siempre es objeto `datetime` en lugar de `int` en algunos casos [`CHANGES.rst:90-91`](../../CHANGES.rst#L90-L91)
- Soporte agregado para rotación de claves. Una lista de claves puede pasarse como `secret_key`, de más antigua a más nueva. La clave más nueva se usa para firmar; todas se prueban para unsign [`CHANGES.rst:92-94`](../../CHANGES.rst#L92-L94)
- Removido el firmante por defecto de respaldo SHA-512 de `default_fallback_signers` [`CHANGES.rst:95-96`](../../CHANGES.rst#L95-L96)
- Se agregó información de tipos para herramientas de escritura estática [`CHANGES.rst:97`](../../CHANGES.rst#L97)

## Versión 1.1.0

**Lanzada:** 26 de octubre de 2018

- Cambio del algoritmo de firma por defecto nuevamente a SHA-1 [`CHANGES.rst:105`](../../CHANGES.rst#L105)
- SHA-512 por defecto agregado como respaldo para usuarios que usaron el lanzamiento retirado 1.0.0 que defaulteaba a SHA-512 [`CHANGES.rst:106-107`](../../CHANGES.rst#L106-L107)
- Soporte agregado para algoritmos de respaldo durante deserialización para permitir cambiar el valor por defecto en el futuro sin romper firmas existentes [`CHANGES.rst:108-110`](../../CHANGES.rst#L108-L110)
- Capitalización de paquetes cambiada nuevamente a minúsculas; el cambio anterior rompió alguna herramienta [`CHANGES.rst:111-112`](../../CHANGES.rst#L111-L112)

## Versión 1.0.0

**Lanzada:** 18 de octubre de 2018

**RETIRADO**

Este lanzamiento fue retirado de PyPI porque cambió el algoritmo por defecto a SHA-512. Esta decisión fue revertida en 1.1.0 y permanece en SHA-1 [`CHANGES.rst:122-124`](../../CHANGES.rst#L122-L124).

- Se descartó soporte para Python 2.6 y 3.3 [`CHANGES.rst:126`](../../CHANGES.rst#L126)
- Refactorización de código de módulo único a paquete. Cualquier objeto en la documentación de API sigue importable desde el nombre `itsdangerous` de nivel superior, pero otras importaciones necesitarán cambios. Futuras versiones removerán muchas importaciones de compatibilidad [`CHANGES.rst:127-130`](../../CHANGES.rst#L127-L130)
- Optimización de cómo se serializan y deserializan marcas de tiempo [`CHANGES.rst:131`](../../CHANGES.rst#L131)
- `base64_decode` lanza `BadData` cuando recibe datos inválidos [`CHANGES.rst:132-133`](../../CHANGES.rst#L132-L133)
- Asegurar que el valor es bytes al firmar para evitar `TypeError` en Python 3 [`CHANGES.rst:134-135`](../../CHANGES.rst#L134-L135)
- Argumento `serializer_kwargs` agregado a `Serializer`, pasado a `dumps` durante `dump_payload` [`CHANGES.rst:136-137`](../../CHANGES.rst#L136-L137)
- Dumps JSON más compacto para strings unicode [`CHANGES.rst:138`](../../CHANGES.rst#L138)
- Usar marca de tiempo completa en lugar de offset, permitiendo fechas anteriores a 2011 [`CHANGES.rst:139-144`](../../CHANGES.rst#L139-L144)
- Detectar carácter `sep` que pueda aparecer en la firma misma y lanzar `ValueError` [`CHANGES.rst:145-146`](../../CHANGES.rst#L145-L146)
- Firma consistente para argumentos con palabras clave en `Serializer.load_payload` en subclases [`CHANGES.rst:147-148`](../../CHANGES.rst#L147-L148)
- Hash intermedio por defecto cambiado de SHA-1 a SHA-512 [`CHANGES.rst:149`](../../CHANGES.rst#L149)
- Conversión de header exp JWS a int al cargar [`CHANGES.rst:150`](../../CHANGES.rst#L150)

## Versión 0.24

**Lanzada:** 28 de marzo de 2014

- Excepción `BadHeader` agregada para encabezados malformados, reemplazando excepción `BadPayload` reutilizada [`CHANGES.rst:158-160`](../../CHANGES.rst#L158-L160)

## Versión 0.23

**Lanzada:** 8 de agosto de 2013

- Corrección de error de empaquetado que causaba que tests y archivos de licencia no se incluyeran [`CHANGES.rst:168-169`](../../CHANGES.rst#L168-L169)

## Versión 0.22

**Lanzada:** 3 de julio de 2013

- Soporte para `TimedJSONWebSignatureSerializer` agregado [`CHANGES.rst:177`](../../CHANGES.rst#L177)
- Posibilidad de sobreescribir función de verificación de firma para permitir implementar algoritmos asimétricos [`CHANGES.rst:178-179`](../../CHANGES.rst#L178-L179)

## Versión 0.21

**Lanzada:** 26 de mayo de 2013

- Corrección de problema en Python 3 que causaba errores inválidos [`CHANGES.rst:187-188`](../../CHANGES.rst#L187-L188)

## Versión 0.20

**Lanzada:** 23 de mayo de 2013

- Corrección de llamada incorrecta a `want_bytes` que rompía algunos usos de ItsDangerous en Python 2.6 [`CHANGES.rst:196-197`](../../CHANGES.rst#L196-L197)

## Versión 0.19

**Lanzada:** 21 de mayo de 2013

- Soporte descartado para 2.5, soporte agregado para 3.3 [`CHANGES.rst:205`](../../CHANGES.rst#L205)

## Versión 0.18

**Lanzada:** 3 de mayo de 2013

- Soporte para JSON Web Signatures (JWS) agregado [`CHANGES.rst:213`](../../CHANGES.rst#L213)

## Versión 0.17

**Lanzada:** 10 de agosto de 2012

- Corrección de error de nombre al sobreescribir método de digest [`CHANGES.rst:221`](../../CHANGES.rst#L221)

## Versión 0.16

**Lanzada:** 11 de julio de 2012

- Posibilidad de pasar valores unicode a `load_payload` para facilitar depuración [`CHANGES.rst:229-230`](../../CHANGES.rst#L229-L230)

## Versión 0.15

**Lanzada:** 11 de julio de 2012

- `load_payload` independiente más robusto lanzando error específico si algo falla [`CHANGES.rst:238-239`](../../CHANGES.rst#L238-L239)
- Refactorización de excepciones para capturar más casos individualmente, atributos agregados [`CHANGES.rst:240-241`](../../CHANGES.rst#L240-L241)
- Corrección de problema que causaba que `load_payload` no funcionara en algunas situaciones con serializadores basados en marca de tiempo [`CHANGES.rst:242-243`](../../CHANGES.rst#L242-L243)
- Método `loads_unsafe` agregado [`CHANGES.rst:244`](../../CHANGES.rst#L244)

## Versión 0.14

**Lanzada:** 29 de junio de 2012

- Refactorización de API para soportar diferentes derivaciones de clave [`CHANGES.rst:252`](../../CHANGES.rst#L252)
- Atributos agregados a excepciones para inspeccionar datos aunque falle verificación de firma [`CHANGES.rst:253-254`](../../CHANGES.rst#L253-L254)

## Versión 0.13

**Lanzada:** 10 de junio de 2012

- Pequeño cambio de API que permite personalización del módulo digest [`CHANGES.rst:262`](../../CHANGES.rst#L262)

## Versión 0.12

**Lanzada:** 22 de febrero de 2012

- Corrección de problema con zona horaria local usada en cálculo de época. Puede invalidar algunas firmas si no se ejecutaba en zona horaria UTC. Revertir a comportamiento antiguo con monkey patching `itsdangerous.EPOCH` [`CHANGES.rst:270-273`](../../CHANGES.rst#L270-L273)

## Versión 0.11

**Lanzada:** 7 de julio de 2011

- Corrección de error de valor no capturado [`CHANGES.rst:281`](../../CHANGES.rst#L281)

## Versión 0.10

**Lanzada:** 25 de junio de 2011

- Refactorización de interfaz para permitir intercambiar serializadores subyacentes pasando módulo en lugar de sobreescribir cargadores y descargadores de payload. Interfaz más compatible con cambios recientes de Django [`CHANGES.rst:289-292`](../../CHANGES.rst#L289-L292)
