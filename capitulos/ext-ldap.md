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

## *Binding*

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

- ***$ldap*** es el objeto conexión.
- ***$base*** es el *DN* base utilizado, es decir, el subárbol donde se realizará la búsqueda.
- ***$filter*** indica un filtro con formato *ldap* para los resultados.
- ***$attributes*** es un *array* que indica los atributos que deseamos que se retornen. El *dn* es siempre retornado. Si no se indica este argumento, se retornan **todos** los atributos de cada coincidencia, lo cual se desaconseja, pues ralentiza mucho la operación.
- ***$attributes_only***: si se establece en 1, solo se retornan los tipos de atributo, no sus valores. Por defecto es 0 (retorna valores).
- ***$sizelimit*** indica el número máximo de entradas devueltas. Un número inferior a 1 significa sin límite, aunque nunca se retornarán más elementos que el máximo definido en el servidor.
- ***$timelimit*** indica el máximo número de segundos que debe tomar la búsqueda. Un número inferior a 1 significa sin límite, aunque nunca tardará más que el máximo definido en el servidor.

La función retorna un objeto del tipo ***LDAP\Result*** (o ***false*** si hay error).

Ejemplo:

```php
$busqueda = ldap_search($ldap, 'DC=dominio,DC=com', '(objectClass=user)', ['cn']);
```

La búsqueda se realiza recursivamente por todos los niveles del subárbol. Si solo se desea un nivel de búsqueda, se debe usar la fucnión `ldap_list()`, que recibe los mismos argumentos que `ldap_search()`. Por otro lado, la función `ldap_read()` también recibe los mismos argumentos, y retorna un resultado que contiene un solo elemento (en este caso, ***$base*** es el *dn* exacto del elemento deseado).

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
- `$entradas[$i]['dn']` es el atributo *dn* de la entrada número ***$i*** (siempre *zero-based*).
- `$entradas[$i]['count']` es el número de atributos de la entrada número ***$i***.
- `$entradas[$i][$j]` el el **nombre** del atributo número ***$j*** de la entrada número ***$i***.
- `$entradas[$i]['atributo']['count']` es el número de valores del atributo ***atributo*** de la entrada número ***$i***.
- `$entradas[$i]['atributo'][$j]` es el valor número ***$j*** del atributo ***atributo*** de la entrada número ***$i***.

Por otro lado, la función `ldap_first_entry()`, con los mismos argumentos, retorna un solo elemento (o ***false*** si hay error). Seguidamente se puede ir usando la función `ldap_next_entry()`, también con los mismos argumentos, que irá retornando la siguiente entrada de la búsqueda, hasta que no haya más (retornará ***false***).

La función `ldap_get_attributes()` recibe como primer argumento el objeto conexión, y como segundo argumento un elemento de una búsqueda.

```php
$ldap = ldap_connect('ldap://servidorldap.com:636');
ldap_bind($ldap);  // binding anónimo
$busqueda = ldap_search($ldap, 'DC=dominio,DC=com', '(objectClass=user)', ['cn']);
$entrada = ldap_first_entry($ldap, $busqueda);
while($entrada)
{
    $atributos = ldap_get_attributes($entrada);
    // tratamos los atributos
    $entrada = ldap_next_entry($ldap, $busqueda);
}
```

El *array* que nos retorna `ldap_get_attributes()` nos proporciona estos elementos:

- `$atributos['count']` es el número de atributos de la entrada.
- `$atributos[0]` es el primer atributo.
- `$atributos[$n]` es el enésimo atributo.
- `$atributos['atributo']['count']` es el número de valores del atributo ***atributo***.
- `$atributos['atributo'][0]` es el primer valor del atributo ***atributo***.
- `$atributos['atributo'][$i]` es el i-ésimo valor del atributo ***atributo***.

### Otras funciones para resultados de búsquedas

ldap_first_attribute
ldap_free_result
ldap_get_dn
ldap_get_values
ldap_next_attribute

## Funciones varias

ldap_error
ldap_get_option
ldap_set_option
