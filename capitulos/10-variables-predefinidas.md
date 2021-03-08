# Variables predefinidas

Las variables *superglobles* son variables predifinidas que están siempre accesibles en cualquier *scope*. No se pueden usar como *variable variables*. Son estas:

`$GLOBALS`, `$_SERVER`, `$_GET`, `$_POST`, `$_FILES`, `$_COOKIE`, `$_SESSION`, `$_REQUEST` y `$_ENV`.

Para acceder a una variable superglobal, no es necesario declararla mediante `global` dentro de una función, ya que están disponibles en todos los *scopes*.

Estas variables son de lectura y escritura.

## $GLOBALS

Es una referencia a todas las variables que están **definidas** en *scope* global.

```php
echo $GLOBALS['foo'];
```

Es una alternativa a declarar `global foo` dentro de una función, pero en esta alternativa seguimos teniendo acceso a una posible variable local ***foo***.

## $_SERVER

Es un *array* que contiene información del entorno (*headers*, rutas, localización de *scripts*, etc.). Algunos de los índices (*strings*) usados para obtener información: `'SERVER_ADDR'`, `'REQUEST_URI'`, `'SERVER_PORT'`, `'HTTPS'`, `'REQUEST_METHOD'`, `'PHP_SELF'`, `'SERVER_PROTOCOL'`, etc.

## $_GET

*Array* asociativo de variables pasadas al *script* como parámetros en la *URL* (en la *query string*). Incluye parámetros de un formulario con método *HTTP GET*. Se accede mediante el nombre del parámetro, y retorna el valor.

## $_POST

*Array* asociativo de variables pasadas al *script* como parámetros de un formulario con método *HTTP POST*. Se accede mediante el nombre del parámetro, y retorna el valor.

## $_FILES

*Array* asociativo de elementos subidos (*uploaded*) al *script* mediante el método *HTTP POST*.

## $_REQUEST

*Array* asociativo que por defecto almacena el contenido de `$_GET`, `$_POST` y `$_COOKIE`.

## $_SESSION

*Array* asociativo que contiene las variables de sesión del *script* actual. El mecanismo de sesión se utiliza mediante las funciones de sesión de *PHP*.

## $_ENV

*Array* asociativo que contiene las variables que recible el *script* del entorno.

## $_COOKIE

*Array* asociativo que contiene las variables que recible el *script* a través de *cookies HTTP*.

## $argc

Contiene el número de elementos que se han pasado por parámetro al *script* actual si se ha ejecutado desde la línea de comandos (incluyendo el nombre del *script*). Equivale a `$_SERVER['argc']`.

## $argv

*Array* que contiene los argumentos que se han pasado al *script* actual si se ha ejecutado desde la línea de comandos (incluyendo el nombre del *script* como primer elemento). Equivale a `$_SERVER['argv']`.
