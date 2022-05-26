# Variables predefinidas

Las variables *superglobles* son variables predifinidas que están siempre accesibles en cualquier *scope*. No se pueden usar como *variable variables*. Son estas:

`$GLOBALS`, `$_SERVER`, `$_GET`, `$_POST`, `$_FILES`, `$_COOKIE`, `$_SESSION`, `$_REQUEST` y `$_ENV`.

Para acceder a una variable superglobal, no hay que declararla mediante `global` dentro de una función, ya que están disponibles en todos los *scopes*.

Estas variables son de lectura y escritura.

Existe una opción importante dentro de la configuración de *PHP* (en ***php.ini***): ***variables_order***. Esta controla cuáles de estas variables superglobales se rellenarán con valores y cuáles no. El valor de esta opción puede contener alguno (o todos) de estos caracteres: ***EGPCS***, que significan:

- ***E*** para la variable ***\$\_ENV***. Es frecuente que no se incluya.
- ***G***: variable ***\$\_GET***.
- ***P***: variable ***\$\_PUT***.
- ***C***: variable ***\$\_COOKIE***.
- ***S***: variable ***\$\_SERVER***.

## $GLOBALS

Es una referencia a todas las variables que están **definidas** en *scope* global.

```php
echo $GLOBALS['foo'];
```

Es una alternativa a declarar `global foo` dentro de una función, pero en esta alternativa seguimos teniendo acceso a una posible variable local ***foo***.

## $_SERVER

Es un *array* que contiene información de variables que **el servidor** ***HTTP*** decide pasar al *script* (*headers*, rutas, localización de *scripts*, etc.). Algunos de los índices (*strings*) usados para obtener información: `'SERVER_ADDR'`, `'REQUEST_URI'`, `'SERVER_PORT'`, `'HTTPS'`, `'REQUEST_METHOD'`, `'PHP_SELF'`, `'SERVER_PROTOCOL'`, etc.

Estas variables están asociadas a una sesión concreta, y no persisten más allá de esta. No se trata de variables de entorno globales del sistema operativo.

## $_GET

*Array* asociativo de variables pasadas al *script* como parámetros en la *URL* (en la *query string*). Incluye parámetros de un formulario con método *HTTP GET*. Se accede mediante el nombre del parámetro, y retorna el valor.

## $_POST

*Array* asociativo de variables pasadas al *script* como parámetros de un formulario con método *HTTP POST*. Se accede mediante el nombre del parámetro, y retorna el valor.

## $_FILES

*Array* asociativo de archivos subidos (*uploaded*) al servidor mediante el método *HTTP POST*. La clave de cada elemento es el nombre del campo en el formulario *HTML*. El contenido de cada elemento es un *array* con los siguientes elementos:

- ***name*** es el nombre original del archivo en la máquina del cliente.
- ***type*** es el tipo *MIME* del archivo, si el navegador proporcionó esta información (no se comprueba en el servidor). Ejemplos podrían ser ***image/gif***, ***image/jpg***, etc.
- ***size*** es el tamaño en *bytes*.
- ***tmp_name*** es el nombre (único) temporal con el cual se ha guardado el archivo en el servidor.
- ***error*** es el código de error asociado a esta subida (si no hay error, es 0).

## $_REQUEST

*Array* asociativo que por defecto almacena el contenido de `$_GET`, `$_POST` y `$_COOKIE`.

## $_SESSION

*Array* asociativo que contiene las variables de sesión del *script* actual. El mecanismo de sesión se utiliza mediante las funciones de sesión de *PHP*.

## $_ENV

*Array* asociativo que contiene las variables que recible el *script* del entorno (sistema operativo). Es frecuente que la opción de configuración ***variables_order*** carezca del carácter ***E***, con lo que con frecuencia nos encontraremos ante un *array* vacío.

## $_COOKIE

*Array* asociativo que contiene las variables que recible el *script* a través de *cookies HTTP*.

## $argc

Contiene el número de elementos que se han pasado por parámetro al *script* actual si se ha ejecutado desde la línea de comandos (incluyendo el nombre del *script*). Equivale a `$_SERVER['argc']`.

## $argv

*Array* que contiene los argumentos que se han pasado al *script* actual si se ha ejecutado desde la línea de comandos (incluyendo el nombre del *script* como primer elemento). Equivale a `$_SERVER['argv']`.
