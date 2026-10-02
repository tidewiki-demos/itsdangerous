# Serializador: Almacenamiento de Estructuras de Datos

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

El serializador proporciona un mecanismo para serializar y deserializar objetos Python complejos con firma criptográfica integrada. Permite guardar y recuperar estructuras de datos de forma segura, verificando que no han sido modificadas. Internamente utiliza JSON por defecto, pero es personalizable.

## Concepto General

[`src/itsdangerous/serializer.py:42-92`](../../src/itsdangerous/serializer.py#L42-L92) La clase `Serializer` envuelve un `Signer` para serializar y firmar datos que no son simplemente bytes. Proporciona métodos `dumps` y `loads` similares al módulo `json`, permitiendo trabajar con estructuras de datos arbitrarias mientras se garantiza su autenticidad.

El flujo fundamental es:
1. **Serialización**: convertir un objeto Python a bytes mediante un serializador (JSON por defecto)
2. **Firma**: firmar esos bytes con el `Signer` para crear una firma criptográfica
3. **Deserialización**: verificar la firma, recuperar los bytes y reconvertir a objeto Python

## Configuración Inicial

[`src/itsdangerous/serializer.py:192-236`](../../src/itsdangerous/serializer.py#L192-L236) El constructor acepta varios parámetros:

- **`secret_key`**: clave secreta para firmar y verificar. Puede ser una lista de claves para soportar rotación de claves.
- **`salt`**: valor extra combinado con la clave secreta para distinguir firmas en diferentes contextos.
- **`serializer`**: objeto con métodos `dumps` y `loads` para serializar datos. Por defecto es JSON.
- **`serializer_kwargs`**: argumentos pasados al método `dumps` del serializador.
- **`signer`**: clase `Signer` a instanciar. Por defecto es `Signer`.
- **`signer_kwargs`**: argumentos al instanciar la clase signer.
- **`fallback_signers`**: lista de configuraciones de firmantes alternativos para intentar al deserializar.

La clase mantiene [`src/itsdangerous/serializer.py:210`](../../src/itsdangerous/serializer.py#L210) una lista `secret_keys` (de más antigua a más nueva) que permite rotación de claves. La más nueva se usa para firmar.

## Serialización de Datos

[`src/itsdangerous/serializer.py:273-278`](../../src/itsdangerous/serializer.py#L273-L278) El método `dump_payload` serializa un objeto usando el serializador interno. Si el serializador retorna texto, se codifica como UTF-8 para obtener bytes.

[`src/itsdangerous/serializer.py:311-322`](../../src/itsdangerous/serializer.py#L311-L322) El método `dumps` realiza la serialización completa: convierte el objeto a bytes mediante `dump_payload`, los firma con un `Signer`, y retorna el resultado codificado como texto (si el serializador interno produce texto) o como bytes.

## Deserialización y Verificación

[`src/itsdangerous/serializer.py:330-345`](../../src/itsdangerous/serializer.py#L330-L345) El método `loads` es el inverso de `dumps`. Convierte la entrada a bytes, itera sobre todos los `Signer` disponibles (el principal más los respaldos), intenta desfirar con cada uno, y si tiene éxito carga el payload. Si ninguno funciona, lanza `BadSignature` con la última excepción.

[`src/itsdangerous/serializer.py:245-271`](../../src/itsdangerous/serializer.py#L245-L271) El método `load_payload` descodifica el payload a partir de bytes. Si el serializador es de texto, primero descodifica los bytes a UTF-8. Maneja excepciones del serializador lanzando `BadPayload`.

[`src/itsdangerous/serializer.py:289-309`](../../src/itsdangerous/serializer.py#L289-L309) El método `iter_unsigners` genera iterativamente los `Signer` a intentar: primero el signer configurado, luego cada uno de los `fallback_signers`. Esto permite graceful degradation si cambian los algoritmos de firma.

## Modo Inseguro para Depuración

[`src/itsdangerous/serializer.py:351-367`](../../src/itsdangerous/serializer.py#L351-L367) El método `loads_unsafe` deserializa sin verificar la firma. Retorna una tupla `(signature_valid, payload)`. Se proporciona solo para depuración y debe usarse solo con serializadores seguros (nunca con pickle).

[`src/itsdangerous/serializer.py:369-397`](../../src/itsdangerous/serializer.py#L369-L397) El método interno `_loads_unsafe_impl` implementa la lógica: intenta primero con `loads` normal; si falla con `BadSignature`, extrae el payload sin verificar y lo carga de todas formas.

## Interfaz de Archivos

[`src/itsdangerous/serializer.py:324-328`](../../src/itsdangerous/serializer.py#L324-L328) El método `dump` es análogo a `dumps` pero escribe directamente a un archivo.

[`src/itsdangerous/serializer.py:347-349`](../../src/itsdangerous/serializer.py#L347-L349) El método `load` es análogo a `loads` pero lee de un archivo.

[`src/itsdangerous/serializer.py:399-406`](../../src/itsdangerous/serializer.py#L399-L406) El método `load_unsafe` es análogo a `loads_unsafe` pero lee de un archivo.

## Detección de Tipo de Serializador

[`src/itsdangerous/serializer.py:35-39`](../../src/itsdangerous/serializer.py#L35-L39) La función `is_text_serializer` detecta si un serializador produce texto o bytes llamando a `dumps({})` y verificando el tipo del resultado. Esto es crucial porque afecta cómo se maneja la codificación al serializar y deserializar.

## Decisiones de Diseño

### Rotación de Claves
[`src/itsdangerous/serializer.py:76-78`](../../src/itsdangerous/serializer.py#L76-L78) En la versión 2.0 se añadió soporte para rotación de claves pasando una lista a `secret_key`. La clave más nueva se usa para firmar, pero todas se aceptan para verificación, permitiendo transiciones seguras entre claves.

### Remoción de Respaldos por Defecto
[`src/itsdangerous/serializer.py:80-82`](../../src/itsdangerous/serializer.py#L80-L82) En la versión 2.0 se removió el fallback SHA-512 por defecto del atributo `default_fallback_signers`. Esto simplifica el comportamiento predeterminado y evita sorpresas con cambios de algoritmo.

## Referencias Relacionadas

- [Firmante: Base de Criptografía](core-signer.md) — el mecanismo de firma subyacente
- [Gestión de Errores](exceptions.md) — excepciones `BadPayload` y `BadSignature`
- [Guía de Serialización](serialization-guide.md) — cómo usar el serializador en práctica
