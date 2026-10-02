# Firmante: Base de Criptografía

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Este módulo implementa la funcionalidad principal de firma criptográfica mediante la clase `Signer`, que genera y verifica firmas HMAC para garantizar que los datos no hayan sido alterados. Es el componente fundamental sobre el cual se construyen características más avanzadas como [Validación con Marca Temporal](timestamp.md) y [Codificación Segura para URL](url-safe.md).

## Componentes principales

### Algoritmos de firma

[`src/itsdangerous/signer.py:15-28`](../../src/itsdangerous/signer.py#L15-L28) define la clase base `SigningAlgorithm`, que establece la interfaz para cualquier algoritmo de firma. Tiene dos métodos:

- `get_signature(key, value)`: genera la firma para una clave y valor dados
- `verify_signature(key, value, sig)`: verifica que una firma sea válida usando `hmac.compare_digest` para evitar ataques de timing

[`src/itsdangerous/signer.py:31-37`](../../src/itsdangerous/signer.py#L31-L37) implementa `NoneAlgorithm`, un algoritmo que no realiza firma alguna (retorna una firma vacía). Se utiliza en contextos donde no se requiere validación.

[`src/itsdangerous/signer.py:48-64`](../../src/itsdangerous/signer.py#L48-L64) implementa `HMACAlgorithm`, que genera firmas usando el algoritmo HMAC. El método de digest utilizado por defecto es SHA-1, pero puede ser configurado en la construcción para usar cualquier función de `hashlib`.

### Clase Signer

[`src/itsdangerous/signer.py:76-112`](../../src/itsdangerous/signer.py#L76-L112) es la clase principal que firma y verifica datos. Requiere:

- **secret_key**: clave secreta (puede ser una lista para rotación de claves)
- **salt**: extra key para distinguir firmas en diferentes contextos (por defecto `b"itsdangerous.Signer"`)
- **sep**: separador entre valor y firma (por defecto `b"."`)
- **key_derivation**: método para derivar la clave de firma a partir de la clave secreta y salt
- **digest_method**: función hash para el HMAC
- **algorithm**: instancia de `SigningAlgorithm` a utilizar

#### Derivación de claves

[`src/itsdangerous/signer.py:182-213`](../../src/itsdangerous/signer.py#L182-L213) el método `derive_key` transforma la clave secreta y salt en una clave de firma usando uno de cuatro métodos:

- `concat`: hash(salt + secret_key)
- `django-concat`: hash(salt + b"signer" + secret_key) (método por defecto)
- `hmac`: HMAC(secret_key, salt)
- `none`: retorna la clave secreta sin transformación

#### Operaciones de firma y verificación

[`src/itsdangerous/signer.py:215-225`](../../src/itsdangerous/signer.py#L215-L225) `get_signature` calcula la firma codificada en base64 para un valor, y `sign` retorna el valor original concatenado con el separador y la firma.

[`src/itsdangerous/signer.py:227-242`](../../src/itsdangerous/signer.py#L227-L242) `verify_signature` verifica que una firma sea válida para un valor dado. Intenta verificar contra todas las claves en `secret_keys` (de más antigua a más nueva), lo que permite rotación de claves.

[`src/itsdangerous/signer.py:244-256`](../../src/itsdangerous/signer.py#L244-L256) `unsign` divide el valor firmado en componente y firma, verifica la firma, y retorna el valor original o lanza `BadSignature` si no es válida.

[`src/itsdangerous/signer.py:258-265`](../../src/itsdangerous/signer.py#L258-L265) `validate` es un método de conveniencia que retorna un booleano indicando si un valor firmado es válido.

## Rotación de claves

[`src/itsdangerous/signer.py:138-142`](../../src/itsdangerous/signer.py#L138-L142) el atributo `secret_keys` es una lista de claves ordenadas de más antigua a más nueva. La clave más nueva (última) se utiliza para firmar nuevos valores, pero la verificación intenta todas las claves en orden inverso. Esto permite mantener múltiples claves válidas durante una transición.

## Restricciones de seguridad

[`src/itsdangerous/signer.py:146-151`](../../src/itsdangerous/signer.py#L146-L151) el separador no puede ser un carácter base64 (letras ASCII, dígitos, `-_=`) porque podría aparecer dentro de la firma misma, lo que crearía ambigüedad al dividir.

[`src/itsdangerous/signer.py:40-45`](../../src/itsdangerous/signer.py#L40-L45) SHA-1 se carga de forma perezosa para soportar builds FIPS que podrían no incluirlo; se falla en tiempo de ejecución con una configuración alternativa en lugar de fallar en tiempo de importación.

## Decisiones de diseño

La clase soporta una lista de claves secretas (desde la versión 2.0) para permitir rotación de claves sin interrumpir la validación de firmas antiguas. La verificación prueba todas las claves en orden inverso, garantizando que los datos firmados con claves anteriores sigan siendo válidos durante la transición.
