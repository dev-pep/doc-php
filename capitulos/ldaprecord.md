# LdapRecord

Esta librería permite trabajar con servidores *LDAP*. Para funcionar, es necesario asegurarse de que se cumplen los requisitos (versión *PHP*, extensión *ldap*).

## Instalación

Desde el directorio de nuestro proyecto, usaremos `composer`:

```
composer require directorytree/ldaprecord
```

La aplicación debe incluir el *autoload* del paquete (que se instala en ***vendor/directorytree/ldaprecord***):

```
require __DIR__ . '/vendor/autoload.php';
```

El *namespace* base del paquete es ***LdapRecord***, y se corresponde con el directorio ***vendor/directorytree/ldaprecord/src***.

## Configuración

Lo primero que hay que hacer es configurar la conexión con el servidor *LDAP*. Para ello debemos crear un objeto de la clase ***LdapRecord\Connection***, al que pasaremos un *array* con los siguientes campos:

- ***hosts***: *array* con 1 o más servidores (IP o nombre), separados por comas. El primero es el principal.
- ***base_dn***: *distinguished name* base. Las búsquedas se harán usando ese nombre como raíz de la búsqueda.
- ***username***: *distinguished name* del usuario para autenticarse en el servidor *LDAP*. Si el servidor es de Directorio Activo, se puede usar el *userPrincipalName* (dirección de correo) o *down-level logon name* (`dominio\\username`).
- ***password***
- ***port***: si no se indica, se usará el puerto por defecto: no *SSL* (389) o *SSL* (636). Si *SSL* está activado y el puerto indicado es 389, se ignorará y se usará 636. Si se habilita *TLS*, el puerto debe ser obligatoriamente 389, y no se pueden usar puertos *SSL*.
- ***use_tls***, ***use_ssl***: se recomienda habilitar una si es posible (se recomienda *TLS*). No se pueden habilitar las dos.
- ***timeout***: en segundos (por defecto 5).
- ***version***: 2 o 3 (recomendado).
- ***follow_referrals***: por defecto ***false***, indica si el servidor de Directorio Activo debe pasar la consulta a otro en caso de no tener la información y saber que el otro servidor la tiene.
- ***options***: opciones para configurar la conexión. Se trata de opciones de la extensión *ldap* de *PHP*. Las constantes ***LDAP_OPT_PROTOCOL_VERSION***, ***LDAP_OPT_NETWORK_TIMEOUT*** o ***LDAP_OPT_REFERRALS*** son ignoradas aquí. Como ejemplo útil, la constante ***LDAP_OPT_X_TLS_REQUIRE_CERT*** especifica cómo proceder con la conexión *TLS* en relación al certificado, y puede tomar uno de estos valores:
    - ***LDAP_OPT_X_TLS_NEVER***: nunca se solicita el certificado. Opción por defecto.
    - ***LDAP_OPT_X_TLS_ALLOW***: se solicita certificado, aunque si no existe o no es correcto, la sesión prosigue normalmente.
    - ***LDAP_OPT_X_TLS_TRY***: se solicita certificado. Si no existe, la sesión prosigue normalmente, pero si el certificado no es correcto, se termina la sesión inmediatamente.
    - ***LDAP_OPT_X_TLS_HARD***, ***LDAP_OPT_X_TLS_DEMAND***: se solicita certificado. Si no existe o no es correcto, la sesión termina inmediatamente.

Solo ***hosts***, ***username*** y ***password*** son obligatorios. Si el servidor admite autenticación anónima, los dos últimos pueden ser ***null***.

## Conexiones

Tras crear la conexión, podemos conectarnos usando el método `connect()`.

```php
$conexion = new Connect([
    'hosts' => ['127.0.0.1'],
    'base_dn' => 'dc=local,dc=com',
    'user' => 'cn=user,dc=local,dc=com',
    'password' => 'secret'
]);
$conexión->connect();
```

El método `connect()` levantará una excepción ***\LdapRecord\Auth\BindException*** en caso de error.

Una vez conectado, el usuario definido en la conexión queda ligado (*bound*) a dicha conexión. Las subsiguiented operaciones sobre el objeto conexión se realizarán bajo este usuario.

Para autenticar un usuario concreto en la página:

```php
if ($connection->auth()->attempt($user, $password))
    echo "Correcto!";
```

Si ***\$user*** es un *distinguished name*, será un *string* del tipo ***cn=user,dc=tesla,dc=com***. En cuanto a ***\$password***, es el *password* de ese usuario (en claro).

Una vez se ha realizado la autenticación, la conexión se vuelve a *bind* al usuario original configurado, a no ser que al método `attempt()` se le pase un tercer argumento ***true***, cosa que dejará *bound* al usuario recién autenticado. Los *bindings* duran solo hasta el fin de la *request*; se deben rehacer a cada *request*.

### *Container*
