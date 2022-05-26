# Extensión ldap

La directiva ***ldap.max_links*** en ***php.ini*** indica el número máximo de conexiones *ldap* por proceso. Por defecto es -1 (ilimitadas).

La secuencia típica de acciones es primero crear una conexión (`ldap_connect()`), luego hacer un *bind* a través de un usuario (`ldap_bind()`), y tras realizar las operaciones deseadas, cerrar la conexión (`ldap_close()`).

Algunos servidores permiten un *bind* anónimo de solo lectura.

En todo caso, para acceder al directorio *ldap* será necesario (a parte del posible usuario para *bind*) disponer de la dirección del servidor y del nombre base (*base name*).

El nombre base es el que se tomará como raíz de las búsquedas. Indicarlo puede ser útil cuando el servidor solo incluye una parte del mundo (una rama concreta, de una provincia, por ejemplo) y no están permitidos los *referrals* (acceso a otros servidores cuando el directorio está compartimentado en dominios distintos).

## Conexión

Lo primero es crear la conexión:

```php
$ldap = ldap_connect($url);
```

La función `ldap_connect()` recibe la dirección del servidor como único parámetro. Esta será del tipo ***ldap://hostname:puerto***, o ***ldaps://hostname:puerto***. La función no conecta en realidad, sino que simplemente comprueba que la sintaxis de la *URL* es correcta, e inicializa los parámetros de conexión. Si falla, retornará ***false***. Si tiene éxito (sintaxis correcta) retornará un objeto de tipo ***LDAP\Connection***.

El *string* puede consistir en varias *URLs* separadas por espacio.

## Binding

Una vez obtenemos el objeto conexión lo pasaremos como primer argumento del *bind* al servidor:

```php
ldap_bind($ldap, $usuario, $password);
```

El usuario, opcional, se pasará como segundo argumento, y contendrá el *distinguished name* del usuario, o cualquier atributo que pueda identificarlo, como *userPrincipalName* o *sAMAccountName* en Directorio Activo.

La contraseña es también opcional. Si se omite, se intentará un *binding* anónimo.

La función `ldap_bind()` retorna ***true*** si tiene éxito y ***false*** en caso contrario.

Una vez hayamos terminado las operaciones que deseamos realizar podemos *unbind* con `ldap_unbind()`, pasándole el objeto conexión como único parámetro. Si lo que queremos es hacer *bind* con un usuario distinto, se debe usar `ldap_bind()`, no usar `ldap_unbind()`, ya que esta última destruye la conexión. La función `ldap_close()` es un *alias* de `ldap_unbind()`.

## Búsqueda

Para realizar búsquedas se usa la función `ldap_search()`

```php
ldap_search($ldap, $base, $filter, $attributes=[], $attributes_only=0, $sizelimit=-1, $timelimit=-1);
```

Solo los tres primeros argumentos son obligatorios:

- ***\$ldap*** es el objeto conexión.
- ***\$base*** es el *DN* base utilizado, es decir, el subárbol donde se realizará la búsqueda.
- ***\$filter*** indica un filtro con formato *ldap* para los resultados.
- ***\$attributes*** es un *array* que indica los atributos que deseamos que se retornen. El *dn* es siempre retornado. Si no se indica este argumento, se retornan **todos** los atributos de cada coincidencia, lo cual se desaconseja, pues ralentiza mucho la operación.
- ***\$attributes_only***: si se establece en 1, solo se retornan los tipos de atributo, no sus valores. Por defecto es 0 (retorna valores).
- ***\$sizelimit*** indica el número máximo de entradas devueltas. Un número inferior a 1 significa sin límite, aunque nunca se retornarán más elementos que el máximo definido en el servidor.
- ***\$timelimit*** indica el máximo número de segundos que debe tomar la búsqueda. Un número inferior a 1 significa sin límite, aunque nunca tardará más que el máximo definido en el servidor.

La función retorna un objeto del tipo ***LDAP\Result*** (o ***false*** si hay error).

Ejemplo:

```php
$busqueda = ldap_search($ldap, 'DC=dominio,DC=com', '(objectClass=user)', ['cn']);
```

La búsqueda se realiza recursivamente por todos los niveles del subárbol. Si solo se desea un nivel de búsqueda, se debe usar la fucnión `ldap_list()`, que recibe los mismos argumentos que `ldap_search()`. Por otro lado, la función `ldap_read()` también recibe los mismos argumentos, y retorna un resultado que contiene un solo elemento (en este caso, ***\$base*** es el *dn* exacto del elemento deseado).

### Filtros *LDAP*

Los filtros *LDAP* tienen su propia sintaxis, y se utilizan principalmente en las búsquedas, aunque pueden tener otros usos. Existen 10 tipos de filtros que veremos a continuación.

#### Filtro de presencia

Sirve para comprobar si una entrada contiene un atributo específico (el atributo tiene por lo menos un valor).

```
(<atributo>=*)
```

Por ejemplo, `(cn=*)` retorna verdadero si la entrada tiene por lo menos un valor ***cn***. El filtro `(objectClass=*)` coincide con todas las entradas, ya que todas deben tener un atributo ***objectClass***.

#### Filtro de igualdad

Comprueba si un atributo concreto de la entrada tiene un valor específico.

```
(<atributo>=<valor>)
```

Por ejemplo, `(givenName=John)`.

La forma de tratar mayúsculas/minúsculas depende de la configuración del servidor.

#### Filtro mayor o igual

Comprueba si un atributo concreto de la entrada tiene **por lo menos un** valor mayor o igual al indicado.

```
(<atributo>>=<valor>)
```

Por ejemplo, `(targetAttribute>=10)`.

El tipo de comparación en números (numérica o alfabética) depende de la configuración del servidor.

#### Filtro menor o igual

Comprueba si un atributo concreto de la entrada tiene **por lo menos un** valor menor o igual al indicado.

```
(<atributo><=<valor>)
```

Por ejemplo, `(targetAttribute<=10)`.

El tipo de comparación en números (numérica o alfabética) depende de la configuración del servidor.

#### Filtro de *substring*

Comprueba si un atributo concreto de la entrada tiene **por lo menos un** valor que encaja con el *substring* indicado.

El *substring* tiene tres partes:

- Componente inicial opcional, el valor debe empezar con este componente.
- Cero o más componentes interiores. Si existe más de uno, deben aparecer en el orden indicado y sin solaparse.
- Componente final opcional, el valor debe terminar con este componente.

Ningún componente interior puede solaparse con un componente inicial o final. Los componentes inicial y final tampoco pueden solaparse.

El formato del *substring* es el siguiente:

- Componente inicial si existe.
- Asterisco (***\****).
- Para cada componente interior:
    - El componente.
    - Asterisco (***\****).
- Componente final si existe.

Ejemplos:

`a*b*cd*e` tiene componente inicial (***a***), componentes interiores (***b*** y ***cd***) y componente final (***e***).

`*b*cd*e` tiene componentes interiores (***b*** y ***cd***) y componente final (***e***).

`a*b*cd*` tiene componente inicial (***a***) y componentes interiores (***b*** y ***cd***).

`*b*cd*` tiene solo componentes interiores (***b*** y ***cd***).

`a*e` tiene componente inicial (***a***) y final (***e***).

El formato del filtro es:

```
(<atributo>=<substring>)
```

Por ejemplo, `(cn=John*)` coincide con un valor de ***cn*** que empiece por ***John***. `(cn=John*son*)` coincide con un valor de ***cn*** que empiece por ***John*** y tenga más adelante el *substring* ***son***.

#### Filtro de coincidencia aproximada

Comprueba si un atributo concreto de la entrada tiene **por lo menos un** valor aproximadamente igual al indicado.

```
(<atributo>~=<valor>)
```

Por ejemplo, `(givenName~=John)`.

Qué se entiende por "aproximadamente igual" depende de la configuración del servidor.

#### Filtro de extensión de coincidencias

Filtro muy flexible, y algo más complejo (se omite explicación).

#### Filto *AND*

Encapsula cero o más filtros precedidos por el carácter *ampersand* (***&***). El resultado es verdadero si todos los filtros se evalúan a verdadero. Es falso si alguno de los filtros se evalúa a falso. Los filtros pueden evaluarse a verdadero, falso o indefinido (por ejemplo en alguna comparación con un tipo no adecuado).

```
(&(<filtro1>)(<filtro2>)...)
```

El filtro *AND* sin filtros en sus interior `(&)` retorna verdadero. Se denomina el *LDAP true*.

#### Filto *OR*

Encapsula cero o más filtros precedidos por el carácter *pipe* o barra vertical (***|***). El resultado es verdadero algún filtro se evalúa a verdadero. Es falso si todos los filtros se evalúan a falso. Como en el caso de *AND*, los filtros interiores pueden evaluarse a verdadero, falso o indefinido.

```
(|(<filtro1>)(<filtro2>)...)
```

El filtro *OR* sin filtros en sus interior `(|)` retorna falso. Se denomina el *LDAP false*.

#### Filtro *NOT*

Sirve para negar un filtro interior, precediéndolo de un signo de exclamación (***!***). Si el filtro interior es verdadero el resultado será falso, y viceversa. Si el filtro interior es indefinido, el resultado será también indefinido.

```
(!(<filtro>))
```

#### Consideraciones sobre filtros

Los filtros pueden *NOT*, *AND* y *OR* pueden anidarse a voluntad y combinarse con todo tipo de filtros.

Algunos caracteres deben indicarse *escaped* cuando nos referimos a una ocurrencia:

- Carácter nulo: ***\\00***
- Paréntesis apertura: ***\\28***
- Paréntesis cierre: ***\\29***
- Asterisco: ***\\2a***
- Contrabarra (*backslash*): ***\\5c***

Entonces, `(cn=*)` coincide con todos los ***cn***, pero `(cn=\2a)` coincide solamente con un ***cn*** que sea igual a un asterisco.

### Trabajar con una búsqueda

El objeto resultado de una búsqueda no se puede utilizar directamente, sino que son necesarias ciertas funciones para trabajar con el resultado.

La función `ldap_count_entries()` recibe como primer argumento el elemento conexión, y como segundo argumento el elemento resultado de la búsqueda. Retorna el número de elementos encontrados en la búsqueda o ***false*** en caso de error.

La función `ldap_get_entries()` recibe los mismos argumentos, y retorna un *array* con los elementos encontrados (o ***false*** si hay error).

```php
$entradas = ldap_get_entries($ldap, $busqueda);
```

En este caso, la organización del *array* retornado es la siguiente:

- `$entradas['count']` es el número de entradas de la búsqueda.
- `$entradas[0]` el la primera de las entradas.
- `$entradas[$i]['dn']` es el atributo *dn* de la entrada número ***\$i*** (siempre *zero-based*).
- `$entradas[$i]['count']` es el número de atributos de la entrada número ***\$i***.
- `$entradas[$i][$j]` el el **nombre** del atributo número ***\$j*** de la entrada número ***\$i***.
- `$entradas[$i]['atributo']['count']` es el número de valores del atributo ***atributo*** de la entrada número ***\$i***.
- `$entradas[$i]['atributo'][$j]` es el valor número ***\$j*** del atributo ***atributo*** de la entrada número ***\$i***.

Los *strings* utilizados como índice se escriben **siempre en minúsculas**.

Por otro lado, la función `ldap_first_entry()`, con los mismos argumentos, retorna un solo elemento (o ***false*** si hay error), del tipo ***\LDAP\ResultEntry***. Seguidamente se puede ir usando la función `ldap_next_entry()`, a la que se le pasará el objeto conexión, y el elemento anterior, retornado por `ldap_first_entry()` o por `ldap_next_entry` (irá retornando la siguiente entrada de la búsqueda); cuando se ha llegado al final, retornará ***false***.

La función `ldap_get_attributes()` recibe como primer argumento el objeto conexión, y como segundo argumento un elemento de una búsqueda (un objeto retornado por `ldap_first_entry()` o `ldap_next_entry()`).

```php
$ldap = ldap_connect('ldap://servidorldap.com:636');
ldap_bind($ldap);  // binding anónimo
$busqueda = ldap_search($ldap, 'DC=dominio,DC=com', '(objectClass=user)', ['cn']);
$entrada = ldap_first_entry($ldap, $busqueda);
while($entrada)
{
    $atributos = ldap_get_attributes($ldap, $entrada);
    // tratamos los atributos
    $entrada = ldap_next_entry($ldap, $busqueda);
}
```

El *array* que nos retorna `ldap_get_attributes()` nos proporciona estos elementos:

- `$atributos['count']` es el número de atributos de la entrada.
- `$atributos[0]` es el nombre del primer atributo.
- `$atributos[$n]` es el nombre del enésimo atributo.
- `$atributos['atributo']['count']` es el número de valores del atributo ***atributo***.
- `$atributos['atributo'][0]` es el primer valor del atributo ***atributo***.
- `$atributos['atributo'][$i]` es el i-ésimo valor del atributo ***atributo***.

### Otras funciones para resultados de búsquedas

Como siempre, y si no se indica lo contrario, el primer parámetro de las siguiente funciones corresponde al objeto retornado por `ldap_connect()`.

Para liberar una búsqueda de memoria:

```php
 ldap_free_result($busqueda)
```

A esta función no se le debe pasar el objeto conexión como primer argumento. El argumento único corresponde al objeto resultado que retorna `ldap_search()`.

En principio no es necesario hacerlo, ya que todos los objetos son liberados de memoria cuando termina el manejo de la *request*, pero si debemos hacer muchas búsquedas, puede ser útil ir liberando memoria.

La función retorna un booleano que indica el éxito o no de la operación.

```php
 ldap_get_dn($ldap, $entrada)
```

Esta función retorna un *string* con el *distinguished name* de la entrada ***\$entrada***, o ***false*** en caso de error.

```php
 ldap_get_values($ldap, $entrada, $atributo)
```

Esta función retorna un *array* con los valores del atributo ***\$atributo*** de la entrada ***\$entrada***. El número de elementos del *array* se puede leer con el índice `['count']`.

## Funciones varias

```php
ldap_error($ldap)
```

Retorna un *string* con el mensaje del último error producido en una operación *LDAP*.

```php
ldap_get_option($ldap, $option, &$value)
ldap_set_option($ldap, $option, $value)
```

Estas funciones sirven para leer o establecer opciones para el funcionamiento de las funciones *LDAP*. Ambas retornan ***true*** si tienen éxito o ***false*** en caso contrario.

El primer argumento, ***\$ldap***, indica la conexión *LDAP* a la que se refieren las opciones leídas/escritas. Estableciendo este argumento a ***NULL***, se refeerirá a opciones globales (para todas las conexiones).

El parámetro ***\$value*** es donde se leerá el valor en caso de *get*. En caso de *set*, será el valor que se establecerá.

La opción a leer/establecer se indicará en ***\$option***, y será de tipo entero. Las opciones puede ser de tipo entero, *array* o *string*. Ejemplos:

```php
ldap_set_option(NULL, LDAP_OPT_PROTOCOL_VERSION, 3);
ldap_set_option(NULL, LDAP_OPT_REFERRALS, 0);
ldap_set_option(NULL, LDAP_OPT_X_TLS_REQUIRE_CERT, LDAP_OPT_X_TLS_ALLOW);
```

### Consideraciones sobre búsquedas

Hay campos que no están indexados para hacer búsquedas. Los servidores establecen índices para los campos más frecuentes. Sin embargo, otros campos no lo están. Veamos un ejemplo. Supongamos que deseamos encontrar los grupos a los que pertenece un usuario.

```php
$grupos = ldap_list($ldap, 'OU=Groups,DC=server,DC=com', '(objectClass=group)', ['cn']);
```

En este caso, obtendríamos todos los grupos de nuestro directorio. Un nodo del tipo grupo (*group*) contiene un campo que se llama ***member*** (en otros servidores puede llamarse de forma parecida), el cual contiene los *distinguished names* de todos los usuarios que pertenecen a tal grupo. Supongamos que deseamos saber cuáles de estos grupos tienen un usuario llamado ***usuario33***. Podríamos pensar que esto solventa el problema:

```php
$groups = ldap_list($ldap, 'OU=Groups,DC=server,DC=com',
    '(&(member=CN=userdp08*)(objectClass=group))', ['cn']);
```

Es decir, queremos todos los grupos que tengan un valor dentro de ***member*** que empiece por ***CN=userdp08***. Sin embargo, el resultado no arroja ningún valor. Simplemente no encuentra tal grupo. Y la razón es porque el campo ***member*** no está indexado por defecto para búsqueda con *substrings*, como sí lo están otros campos más habituales (el administrador del servidor puede indexar los campos que desee).

Por lo tanto, el único remedio aquí es pasarle al filtro el valor del *distinguished name* **completo** del usuario a buscar.

Por otro lado, la mejor forma de hacer esta búsqueda es al revés, a partir del registro del usuario, y no del grupo. En este caso, leeríamos el contenido del atributo ***memberOf*** del usuario, en lugar de leer todos los miembros de todos los grupos. En este caso, el atributo ***memberOf*** de usuario es un ***backlink*** (vínculo de retroceso), ya que para mantener la integridad, el vínculo es bidireccional, es decir el grupo apunta al usuario miembro a través de ***member***, mientras el miembro (usuario) apunta al grupo a través de ***memberOf***. Podríamos decir que los ***backlinks*** son enlaces redundantes.

Como curiosidad, ***objectCategory*** es un campo indexado por defecto, pero no ***objectClass***.
