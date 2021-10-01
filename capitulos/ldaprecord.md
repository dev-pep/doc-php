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
- ***username***: *distinguished name* completo del usuario para autenticarse en el servidor *LDAP*. Si el servidor es de Directorio Activo, se pueden usar otros campos como el *userPrincipalName* o el *sAMAccountName*.
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

En ocasiones, si tenemos problemas con la autenticación relacionados con el certificado, se puede solucionar editando el archivo de configuración de *LDAP* en el servidor *web* llamado ***ldap.conf*** (en *Linux* suele estar en ***/etc/ldap/***). Si no existe habrá que crearlo. En todo caso, se puede añadir la línea:

```
TLS_REQCERT never
```

En este caso se ignorarán certificados inexistentes o inválidos. Si así funciona, una alternativa más segura es copiar un certificado AC válido a nuestro servidor *web*, y cambiar el ***ldap.conf*** para que lo use siempre:

```
TLS_CACERT /ruta/al/certificado.pem
TLS_REQCERT hard
```

> Cada vez que se haga un cambio en el archivo ***ldap.conf*** hay que reiniciar el servidor *web*.

Una vez conectado, el usuario definido en la conexión queda ligado (*bound*) al servidor. Las subsiguientes operaciones realizadas con esa conexión en el servidor se realizarán bajo este usuario.

Para autenticar un usuario concreto en la página:

```php
if ($conexion->auth()->attempt($user, $password))
    echo "Correcto!";
```

El método `auth()` retorna un objeto de tipo ***\Ldaprecord\Auth\Guard***.

Si ***\$user*** es un *distinguished name*, será un *string* del tipo ***cn=user,dc=tesla,dc=com***. En cuanto a ***\$password***, es el *password* de ese usuario (en claro).

Una vez se ha realizado la autenticación, la conexión se vuelve a *bind* al usuario original configurado, a no ser que al método `attempt()` se le pase un tercer argumento ***true***, cosa que dejará *bound* al usuario recién autenticado. Los *bindings* duran solo hasta el fin de la *request*; se deben rehacer a cada *request*.

### Contenedor de conexiones

Las conexiones se pueden añadir a un contenedor (de la clase ***\Ldaprecord\Container***), mediante el método estático `addConnection()`, al que se le pasa el objeto conexión. Si este no se ha conectado todavía al servidor (mediante el método `connect()`, por ejemplo), se conectará automáticamente cuando invoquemos cualquier operación con el servidor *LDAP*.

Al añadir la conexión al contenedor, debemos especificar un nombre como segundo argumento, de tal manera que sea fácil extraerlas del contenedor. Si no se especifica nombre, se añadirá con el nombre ***default***. Si un nombre ya existe, se sobrescribirá la conexión.

```php
use \Ldaprecord\Container;

Container::addConnection($conexion, 'con-1');
```

El contenedor permite trabajar con varios servidores *LDAP*.

[MODELS???????]

Para obtener una conexión del contenedor, se usa el método estático `getConnection()`, pasándole como argumento el nombre de la misma. Si lo que queremos es obtener la conexión por defecto (la que tiene nombre ***default***), usaremos `getDefaultConnection()` sin argumentos.

Para cambiar el nombre que se usará para la conexión por defecto, usaremos `setDefaultConnection()` con el nombre nuevo.

Para comprobar si existe una conexión con un nombre concreto, pasaremos ese nombre al método `exists()`, teniendo en cuenta que este no es un método estático, con lo cual deberemos usar una instancia del contenedor (que obtendremos mediante el método estático `getInstance()`):

```php
if(Container::getInstance()->exists('nombre-con'))
{ /* ... */ }
```

## Búsquedas

### *Query builder*

Es posible realizar consultas al servidor *LDAP* mediante *query builders* (de la clase ***Ldaprecord\Query\Builder***). Es posible obtener un objeto de este tipo a través del método `query()` del objeto conexión. Los métodos que dan forma al *query builder* retornan a su vez un *query builder*, con lo que se pueden ir encadenando. Finalmente, para convertirlo en una colección, se usa el método `get()`. En este caso, la colección será un simple *array*, cuyos elementos serán los registros del servidor *LDAP* (cada registro es a su vez un *array* de campos).

El método `select()` selecciona los campos que se retornarán. Se le debe pasar un *array* con los nombres de los campos, o una cantidad arbitraria de argumentos, con los nombres deseados.

El método `find()` retorna un solo registro (si lo encuentra), o ***null***. Este registro es un *array* con los campos del registro. Para ello, hay que pasarle al método el *distinguished name* deseado. Si queremos que levante una excepción (***\Ldaprecord\Models\ModelNotFoundException***) en caso de no encontrar el registro, usaremos `findOrFail()`.

Si deseamos buscar por un atributo distinto al *distinguished name*, usaremos `findBy()`, cuyo primer argumento será el nombre del campo deseado, y el segundo el valor a buscar.

Para obtener solo el primer registro de una consulta, se usará `first()`. Si queremos que levante excepción si la consulta está vacía, se usará `firstOrFail()`.

Para limitar el número de registros, se debe pasar tal número al método `limit()`.

También es posible aplicar cláusulas *where* mediante el método `where()`, que tiene tres parámetros *string*: el primero es el nombre del campo, el segundo el operador, y el tercero el valor a comparar. Si el operador es el de igualdad, se puede obviar.

Si queremos aplicar varias condiciones en un solo método `where()`, podemos pasarle un *array*:

```php
$registros = $query->where('nombre', 'Pepe')
    ->where('ciudad', 'Sabadell')
    ->where('edad', '35')
    ->get();
```

Lo anterior equivale a:

```php
$filtros = [
    'nombre' => 'Pepe',
    'ciudad' => 'Sabadell',
    'edad' => '35'
];

$registros = $query->where($filtros)
    ->get();
```

Esto es válido cuando todas las condiciones usan el operador igualdad, pero si no es así, el *array* tendrá otro formato:

```php
$filtros = [
    ['nombre', '=', 'Pepe'],
    ['ciudad', '=', 'Sabadell'],
    ['edad', '<=', '35']
];

$registros = $query->where($filtros)
    ->get();
```

De forma similar, para aplicar una cláusula *or where*, se puede usar el método `OrWhere()`.

Para restringir la consulta a partir de un nodo del árbol, se indicará un *distinguished name* base:

```php
$registros = $query->in('ou=Accounting,dc=tesla,dc=com')
    ->where(/*...*/)
    ->get();
```

A la hora de especificar este nombre base, podemos sustituir directamente el nombre base configurado en la conexión (***base_dn***) dentro de cualquier *query*:

```php
$registros = $query->in('ou=Accounting,{base}')->...
```
