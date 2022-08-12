# LdapRecord

Esta librería permite trabajar con servidores *LDAP*. Para funcionar, es necesario asegurarse de que se cumplen los requisitos (versión *PHP*, extensión *ldap*).

## Instalación

Desde el directorio de nuestro proyecto, usaremos `composer`:

```
composer require directorytree/ldaprecord
```

La librería se instala en ***vendor/directorytree/ldaprecord***. Nuestra aplicación debe incluir el *autoload* de los paquetes de *composer*:

```
require __DIR__ . '/vendor/autoload.php';
```

El *namespace* base del paquete es ***LdapRecord***, y se corresponde con el directorio ***vendor/directorytree/ldaprecord/src***.

## Configuración

Lo primero que hay que hacer es configurar la conexión con el servidor *LDAP*. Para ello debemos crear un objeto de la clase ***LdapRecord\\Connection***, al que pasaremos un *array* con los siguientes campos:

- ***hosts***: *array* con 1 o más servidores (IP o nombre), separados por comas. El primero es el principal.
- ***base_dn***: *distinguished name* (*DN*) base. Las búsquedas se harán usando ese nombre como raíz de la búsqueda. Si solo deseamos usar el servidor *LDAP* para autenticación, este valor no es necesario.
- ***username***: *DN* completo del usuario para autenticarse en el servidor *LDAP*. Si el servidor es de Directorio Activo, se pueden usar otros campos como el *userPrincipalName* o el *sAMAccountName*.
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
$conexion->connect();
```

El método `connect()` levantará una excepción ***LdapRecord\\Auth\\BindException*** en caso de error.

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

El método `auth()` retorna un objeto de tipo ***Ldaprecord\\Auth\\Guard***.

Si ***\$user*** es un *DN*, será un *string* del tipo ***cn=user,dc=tesla,dc=com***. En cuanto a ***\$password***, es el *password* de ese usuario (en claro).

Una vez se ha realizado la autenticación, la conexión se vuelve a *bind* al usuario original configurado, a no ser que al método `attempt()` se le pase un tercer argumento ***true***, cosa que dejará *bound* al usuario recién autenticado. Los *bindings* duran solo hasta el fin de la *request*; se deben rehacer a cada *request*.

### Contenedor de conexiones

Las conexiones se pueden añadir a un contenedor (de la clase ***Ldaprecord\\Container***), mediante el método estático `addConnection()`, al que se le pasa el objeto conexión. Si este no se ha conectado todavía al servidor (mediante el método `connect()`, por ejemplo), se conectará automáticamente cuando invoquemos cualquier operación con el servidor *LDAP*.

Al añadir la conexión al contenedor, debemos especificar un nombre como segundo argumento, de tal manera que sea fácil extraerlas del contenedor. Si no se especifica nombre, se añadirá con el nombre ***default***. Si un nombre ya existe, se sobrescribirá la conexión.

```php
use Ldaprecord\Container;

Container::addConnection($conexion, 'con-1');
```

El contenedor permite trabajar con varios servidores *LDAP*. Por otro lado, permite trabajar con modelos (ver más adelante).

Para obtener una conexión del contenedor, se usa el método estático `getConnection()`, pasándole como argumento el nombre de la misma. Si lo que queremos es obtener la conexión por defecto (la que tiene nombre ***default***), usaremos `getDefaultConnection()` sin argumentos.

Para cambiar el nombre que se usará para la conexión por defecto, usaremos `setDefaultConnection()` con el nombre nuevo.

Para comprobar si existe una conexión con un nombre concreto, pasaremos ese nombre al método `exists()`, teniendo en cuenta que este no es un método estático, con lo cual deberemos usar una instancia del contenedor (que obtendremos mediante el método estático `getInstance()`):

```php
if(Container::getInstance()->exists('nombre-con'))
{ /* ... */ }
```

## Búsquedas

### Query builder

Es posible realizar consultas al servidor *LDAP* mediante *query builders* (de la clase ***Ldaprecord\\Query\\Builder***). Es posible obtener un objeto de este tipo a través del método `query()` del objeto conexión. Los métodos que dan forma al *query builder* retornan a su vez un *query builder*, con lo que se pueden ir encadenando. Finalmente, para convertirlo en una colección, se usa el método `get()`. En este caso, la colección será un simple *array*, cuyos elementos serán los registros del servidor *LDAP* (cada registro es a su vez un *array* de campos).

El método `select()` selecciona los campos que se retornarán. Se le debe pasar un *array* con los nombres de los campos, o una cantidad arbitraria de argumentos, con los nombres deseados.

El método `find()` retorna un solo registro (si lo encuentra), o ***null***. Este registro es un *array* con los campos del registro. Para ello, hay que pasarle al método el *DN* deseado. Si queremos que levante una excepción (***Ldaprecord\\Models\\ModelNotFoundException***) en caso de no encontrar el registro, usaremos `findOrFail()`.

Si deseamos buscar por un atributo distinto al *DN*, usaremos `findBy()`, cuyo primer argumento será el nombre del campo deseado, y el segundo el valor a buscar.

Para obtener solo el primer registro de una consulta, se usará `first()`. Si queremos que levante excepción si la consulta está vacía, se usará `firstOrFail()`.

> Es importante tener en cuenta que los campos de un directorio *Ldap* son multivalor por naturaleza, lo cual significa que son siempre un *array* de valores.

Para limitar el número de registros, se debe pasar tal número al método `limit()`.

También es posible aplicar cláusulas *where* mediante el método `where()`, que tiene tres parámetros *string*: el primero es el nombre del campo, el segundo el operador, y el tercero el valor a comparar. Si el operador es el de igualdad, se puede obviar.

Si queremos aplicar varias condiciones en un solo método `where()`, podemos pasarle un *array*:

```php
$query = $conexion->query();
$registros = $query->where('name', 'Pepe')
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

De forma similar, para aplicar una cláusula *or where*, se puede usar el método `orWhere()`.

Para restringir la consulta a partir de un nodo del árbol, se indicará un *DN* base:

```php
$registros = $query->in('ou=Accounting,dc=tesla,dc=com')
    ->where(/*...*/)
    ->get();
```

A la hora de especificar este nombre base, podemos sustituir directamente el nombre base configurado en la conexión (***base_dn***) dentro de cualquier *query*:

```php
$registros = $query
    ->in('ou=Accounting,dc=tesla,dc=com')
    ->...
```

Para acceder a los campos de un registro, se accederá indexando el *array* con el nombre del campo, y se tendrá en cuenta que el valor retornado es un *array*:

```php
$usuario = $query->find('cn=username,dc=tesla,dc=com');
echo $usuario['name'][0];
```

### Otros métodos

Entre los numerosos métodos disponibles para los *query builders* podemos destacar:

- `delete()` elimina la entrada del servidor *LDAP*.
- `getConnection()` retorna la conexión sobre la que está funcionando el *query builder*.
- `getDn()` retorna el *DN* de la conexión que está usando el *query builder*.
- `insert()` inserta un nuevo registro. El primer argumento debe ser el *DN*, y el segundo un *array* con los nombres de los campos como claves, y sus correspondientes valores (si el valor es múltiple, será, a su vez un *array*).
- `paginate()` se usa cuando el número de registros retornados puede exceder el límite (normalmente 1000).

## Modelos

*LdapRecord* dispone de modelos *ORM*. Como es habitual, un modelo se corresponde con un tipo objeto *LDAP* en el directorio. Para trabajar con modelos es obligatorio incluir por lo menos una conexión en el contenedor. Existen ya modelos incluidos para servidores *LDAP* populares (como Directorio Activo, o *OpenLDAP*). Pero podemos crear nuestros propios modelos.

### Definición de modelos

Los modelos *ORM* de *LdapRecord* extienden la clase ***Ldaprecord\\Models\\Model***. Los modelos correspondientes a Directorio Activo están en el *namespace* ***Ldaprecord\\Models\\ActiveDirectory***. El modelo ***Entry*** sirve para obtener todo tipo de entradas del directorio. Otros modelos sirven para obtener un tipo concreto de entrada: ***User***, ***Group***, ***Computer***, ***OrganizationalUnit***, ***Printer***, etc. Los modelos para *OpenLDAP* están en el *namespace* ***LdapRecord\\Models\\OpenLDAP***, etc.

Por defecto, los modelos intentan usar la conexión por defecto del contenedor. Si se desea que no sea así, el modelo debe definir una propiedad `protected` ***\$connection*** con el nombre de la conexión deseada.

Para obtener *distinguished names* del modelo, este dispone de los siguientes métodos:

- `getDn()` retorna el *DN* completo.
- `getRdn()` retorna el *DN* relativo (al nombre base).
- `getParentDn()` retorna el *DN* completo del objeto padre.
- `getName()` retorna el nombre del objeto.

Para definir valores por defecto en nuestro modelo, definiremos una propiedad ***\$attributes*** como un *array* en el que las claves serán los nombres de los campos, y los valores serán los que deseamos que tengan por defecto. Dado que, como hemos visto, los campos *LDAP* son multivalor por naturaleza, los valores serán siempre un *array*, aunque se trate de un único valor.

### Obtener valores

La siguiente sentencia retornaría **todos** los usuarios del directorio (si no exceden el límite):

```php
use Ldaprecord\Models\ActiveDirectory\User;

$registros = User::get();
```

Podemos utilizar como método inicial uno de los definidos para *query builders*, con lo que lo podremos ir refinando, igual que si de un *query builder* se tratara. En este caso, se trata de un objeto del tipo ***Ldaprecord\\Query\\Model\\ActiveDirectoryBuilder***. Al final de la cadena, `get()` retornará los registros en una colección del tipo ***Ldaprecord\\Models\\Collection***.

A parte de las restricciones habituales de los *query builders*, estos modelos disponen de restricciones específicas de los directorios *LDAP*: `ancestors()`, `siblings()` y `descendants()`.

Si queremos ver si nuestro modelo ha cambiado en el servidor, tenemos el método `fresh()` que retornará el estado actual del modelo, pero no afectará a la instancia actual del modelo. Si deseamos que la instancia actual se refresque con los datos del servidor, usaremos el método `refresh()`.

### Obtención de un único modelo

Algunos métodos que retornan un único modelo (no un *query builder* ni un colección):

- Primer registro: `first()`
- Búsqueda por *DN*: `find($distinguishedName)`
- Búsqueda por cualquier atributo: `findBy($attributeName, $attributeValue)`

Si queremos que se levente excepción al no encontrar una coincidencia (***Ldaprecord\\Models\\ModelNotFoundException***), usaremos los métodos correspondientes: `firstOrFail()`, `findOrFail()`, `findByOrFail()`.

### Creación y actualización de modelos

Para la creación de un objeto *LDAP* requiere siempre un *DN* completo. Si no dispone de él, para su inserción en el directorio intentará construir un *DN* a partir de los datos de que disponga, y del *DN* base definido. Algunos objetos precisan, además, otros campos adicionales obligatorios: un usuario necesita también el campo ***cn*** (*common name*), una *organizational unit* necesita por lo menos un campo ***ou***, etc.

La inserción de un nuevo objeto se hace a través del método `save()`:

```php
use Ldaprecord\Models\ActiveDirectory\User;

$usuario = new User();
$usuario -> cn = 'John Doe';
$usuario -> save();  // se generará un DN para el objeto
```

Para definir el nodo padre donde se colocará el nuevo objeto, debemos definir el *DN* base que se usará para crear su *DN*:

```php
$usuario = new User();
$usuario -> cn = 'John Doe';
$usuario -> inside('ou=Ventas,dc=tesla,dc=com');
$usuario -> save();
```

En el ejemplo, no se usa el *base DN* definido, sino el que indicamos en `inside()`. Este método no solo acepta *strings*; también se le puede pasar una instancia de un modelo que contenga, por ejemplo, un nodo *organizational unit*.

Alternativamente, podemos definir el *DN* explícitamente mediante el método `setDn()`.

Para **actualizar un modelo ya existente**, una vez lo hemos obtenido y cambiado los campos deseados, se llama a `save()` de la forma habitual.

Es posible **mover un objeto** a un nodo distinto del árbol. Para ello usaremos el método `move()` sobre la instancia del modelo, al que le pasaremos como primer argumento la nueva ubicación (un *DN* base, o una instancia de organizational unit, por ejemplo). En este caso, el *DN* del objeto será cambiado para reflejar su nueva ubicación.

Para **renombrar** un objeto, se usa el método `rename()` en la instancia del modelo, al que le pasaremos el nuevo *relative distinguished name* (*DN* sin el *base DN*).

### Acceso a los atributos

> El nombre de los atributos en *case-insensitive*. Si indicamos guión bajo (no aceptado en *LDAP*) en un nombre de atributo, será cambiado por guión automáticamente.

Para acceder a los valores de los campos, una vez hayamos obtenido nuestro modelo, se accederá mediante sintaxis de atributo, teniendo siempre en cuenta la naturaleza multivalor de los campos de *Ldap*, es decir, teniendo en cuenta que siempre obtendremos un *array* de valores.

```php
use Ldaprecord\Models\ActiveDirectory\User;

$registros = User::in('ou=Marketing,dc=tesla,dc=com')
                 ->first();
$nombre = $registros->name[0];
```

Existen una serie de métodos que (aplicados a una instancia del modelo) permiten el acceso a los atributos.

El método `getAttributes()` retorna un *array* con **todos** los atributos. Las claves del *array* son los nombres de atributos, y los valores son **siempre** un *array* de 0 o más valores (los campos de *LDAP* son multivalor).

Para obtener el valor (*array*) de un solo atributo: `getAttribute()`, al que se le debe pasar el nombre del campo deseado. Si en lugar de un *array* deseamos recibir solo el primer valor, usaremos `getFirstAttribute()`, que retornará un *string*, o *null*.

Si queremos saber si existe una determinada clave entre los atributos: `hasAttribute()`, pasándole el nombre del atributo.

Para **añadir** un valor a un atributo (sin eliminar los anteriores): `addAttributeValue()`, pasándole el nombre del atributo y el valor deseado (todo *strings*).

`countAttributes()` retorna el número de atributos que contiene el modelo.

Al dar valor a un atributo directamente, se le debe dar un *array*. Si no, el valor se convertirá automáticamente a un *array* de un solo elemento. También se puede usar el método `setFirstAttribute()` (al que se le pasa el nombre del atributo y el valor deseado), que dará valor al primer atributo del campo, aunque no exista.

### Eliminación de objetos

Para eliminar un objeto se usará el método `delete()` sobre una instancia del modelo.

Alternativamente, en lugar de usar una instancia, se puede usar el método estático `destroy()` sobre la clase del modelo. Este debe recibir un *string* con un *DN*, o un *array* de ellos.

Si un objeto contiene otros objetos, no será eliminado. Si queremos realizar una eliminación recursiva, tanto `delete()` como `destroy()` aceptan un segundo parámetro ***true*** que permitirá tal eliminación.

### Comparación de modelos

Existen varios métodos para comprobar la igualdad o relación entre dos modelos.

`$user1->is($user2)` retorna verdadero si son el mismo objeto del directorio.

Otros métodos: `isDescendantOf()`, `isAncestorOf()`, `isChildOf()`, `isParentOf()`.

### Accessors y mutators

> El mecanismo es casi idéntico al de los *accessors* y *mutators* de *Eloquent ORM*.

Definiendo este tipo de métodos modificamos los valores de los atributos de un modelo.

Un *accessor* modifica un atributo al leerlo del directorio. El nombre del método debe ser del tipo ***get*** + ***\<nombreatributo>*** + ***Attribute***. Recordemos que los nombres de atributos son *case-insensitive* en *LDAP*.

```php
use Ldaprecord\Models\Model;

class Usuario extends Model
{
    public function getFotoperfilAttribute($valor)
    {
        /* ... */
        return $valor_modificado;
    }
}
```

En este caso, cada vez que accedamos al atributo ***fotoPerfil*** (o cualquier otra combinación de mayúsculas/minúsculas), recibiremos el valor que retorne este *accessor*. Debe tenerse en cuenta, que el valor que recibirá el método (***\$valor***) será siempre un *array*, aunque no tiene por qué retornar un *array*.

Las mayúsculas en el nombre de un *accessor* indican guiones en el nombre del atributo. Por ejemplo, un *accessor* denominado ***getFotoPerfilAttribute*** se relacionaría con el atributo ***foto-perfil*** (o cualquier otra combinación de mayúsculas/minúsculas).

Un *mutator* hace lo contrario: se usa para modificar un valor destinado al directorio. En este caso, en lugar de retornar un valor, modifica el modelo adecuadamente, cambiando la entrada correspondiente del atributo ***\$attributes***:

```php
use Ldaprecord\Models\Model;

class Usuario extends Model
{
    public function setFotoperfilAttribute($valor)
    {
        /* ... */
        $this->attributes['fotoPerfil'] =
            [$valor_modificado];
    }
}
```

En este caso sí debemos enviar un *array* como valor para el directorio.

La nomenclatura de un *mutator* sigue el mismo esquema que un *accessor*, cambiando ***get*** por ***set***.

Existen una serie de mapeos predeterminados entre datos *PHP* de/a *LDAP*. Esto se define en el atributo ***\$casts*** de la clase del modelo. Se le debe dar como valor un *array* donde las claves son el nombre del campo y los valores el tipo al/del que se debe convertir el campo (***integer***, ***float***, ***string***, ***boolean***, etc.).

## Autenticación

*LdapRecord* no establece ningún tipo de sesión que persista para otras *requests*.

Para autenticación se usará el método `attempt()` visto anteriormente. Veamos un ejemplo que recogería la *request* de una pantalla de *login*:

```php
$connection = new \LdapRecord\Connection([
    /* datos de la conexión */
]);
$connection->connect();
$user = $connection->query()
    ->where('samaccountname', '=', $_POST['username'])
    ->first();

if ($connection->auth()
    ->attempt($user['distinguishedname'][0],
              $_POST['password']))
{ /* OK */ }
else
{ /* Username y/o password incorrectos */ }
```

Para restringir privilegios, se deben obtener los grupos a los que pertenece el usuario. En este caso, leeríamos el valor de la clave ***memberof***, que retornará un *array* de *strings* con todos los grupos a los que pertenece el usuario.

Para restringir según pertenencia a una *organizational unit*, se puede restringir la búsqueda del usuario a dicha *OU* para ver si existe allí.
