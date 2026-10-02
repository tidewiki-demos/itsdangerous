# Conceptos Fundamentales

<!-- Maintained by Tidewiki. Edits are kept on later updates; wrap text in tidewiki:keep markers to freeze it. -->

Este documento explica los principios detrás de la firma criptográfica, la serialización y el diseño general del sistema ItsDangerous.

## Firmante vs Serializador

ItsDangerous proporciona dos niveles de manejo de datos. El [Firmante](core-signer.md) es el sistema básico que firma un valor `bytes` según los parámetros de firma especificados. El [Serializador](serializer.md) envuelve un firmante para permitir serializar y firmar otros tipos de datos además de `bytes`. [`docs/concepts.rst:5-11`](../../docs/concepts.rst#L5-L11)

En la mayoría de los casos, querrás usar un serializador en lugar de un firmante directamente. Puedes configurar los parámetros de firma a través del serializador e incluso proporcionar firmantes de respaldo para actualizar tokens antiguos a nuevos parámetros. [`docs/concepts.rst:13-15`](../../docs/concepts.rst#L13-L15)

## La Clave Secreta

Las firmas se aseguran mediante la `secret_key`. Típicamente se usa una única clave secreta con todos los firmantes, y la sal se utiliza para distinguir diferentes contextos. Cambiar la clave secreta invalidará todos los tokens existentes. [`docs/concepts.rst:18-23`](../../docs/concepts.rst#L18-L23)

Debe ser una cadena larga de bytes aleatoria. Este valor debe mantenerse secreto y no debe guardarse en el código fuente ni confirmarse en el control de versiones. Si un atacante aprende la clave secreta, puede cambiar y refirmar datos para que parezcan válidos. Si sospechas que esto sucedió, cambia la clave secreta para invalidar los tokens existentes. [`docs/concepts.rst:25-29`](../../docs/concepts.rst#L25-L29)

Una forma de mantener la clave secreta separada es leerla de una variable de entorno. Al implementar por primera vez, genera una clave y establece la variable de entorno al ejecutar la aplicación. Todos los gestores de procesos (como systemd) y servicios de alojamiento tienen una forma de especificar variables de entorno. [`docs/concepts.rst:31-35`](../../docs/concepts.rst#L31-L35)

Una forma de generar una clave es utilizar `os.urandom`. [`docs/concepts.rst:49`](../../docs/concepts.rst#L49)

## La Sal

La sal se combina con la clave secreta para derivar una clave única que distingue diferentes contextos. A diferencia de la clave secreta, la sal no tiene que ser aleatoria y puede guardarse en el código. Solo debe ser única entre contextos, no privada. [`docs/concepts.rst:59-62`](../../docs/concepts.rst#L59-L62)

Por ejemplo, si deseas enviar enlaces de activación para activar cuentas de usuario y enlaces de actualización para actualizar usuarios a cuentas de pago, y solo firmas el ID del usuario sin usar sales diferentes, un usuario podría reutilizar el token del enlace de activación para actualizar la cuenta. Si usas sales diferentes, las firmas serán diferentes y no serán válidas en el otro contexto. [`docs/concepts.rst:64-69`](../../docs/concepts.rst#L64-L69)

El serializador con la misma sal puede cargar los datos, pero otros no. Esto proporciona aislamiento seguro entre diferentes usos de los tokens. [`docs/concepts.rst:93-98`](../../docs/concepts.rst#L93-L98)

## Rotación de Claves

La rotación de claves puede proporcionar una capa extra de mitigación contra un atacante que descubra una clave secreta. Un sistema de rotación mantiene una lista de claves válidas, generando una clave nueva y eliminando la más antigua periódicamente. Si toma cuatro semanas que un atacante descifre una clave, pero la clave se rota después de tres semanas, no podrá usar ninguna clave que descifre. Sin embargo, si un usuario no actualiza su token dentro de tres semanas, también será inválido. [`docs/concepts.rst:104-110`](../../docs/concepts.rst#L104-L110)

El sistema que genera y mantiene esta lista está fuera del alcance de ItsDangerous, pero ItsDangerous sí admite validar contra una lista de claves. [`docs/concepts.rst:112-114`](../../docs/concepts.rst#L112-L114)

En lugar de pasar una única clave, puedes pasar una lista de claves de más antigua a más nueva. Al firmar se usa la última (más nueva) clave, y al validar se prueba cada clave de más nueva a más antigua antes de generar un error de validación. [`docs/concepts.rst:116-119`](../../docs/concepts.rst#L116-L119)

## Seguridad del Método Digest

Un firmante se configura con un `digest_method`, una función hash que se utiliza como paso intermedio al generar la firma HMAC. El método predeterminado es SHA-1. Ocasionalmente, los usuarios se preocupan por este valor predeterminado porque han oído hablar de colisiones de hash con SHA-1. [`docs/concepts.rst:140-144`](../../docs/concepts.rst#L140-L144)

Cuando se usa como paso intermedio iterado en HMAC, SHA-1 no es inseguro. De hecho, incluso MD5 sigue siendo seguro en HMAC. La seguridad del hash por sí solo no se aplica cuando se usa en HMAC. [`docs/concepts.rst:146-148`](../../docs/concepts.rst#L146-L148)

Si un proyecto considera SHA-1 un riesgo de todas formas, puede configurar el firmante con un método digest diferente, como SHA-512. Un firmante de respaldo para SHA-1 puede configurarse para que los tokens antiguos se actualicen. SHA-512 produce un hash más largo, por lo que los tokens ocuparán más espacio, lo cual es relevante en cookies y URLs. [`docs/concepts.rst:150-154`](../../docs/concepts.rst#L150-L154)
