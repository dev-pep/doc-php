# Otros

## Autenticación *HTTP*

El servidor puede estar configurado para solicitar automáticamente autenticación para visitar un sitio *web*. En ese caso, al autenticarse, el servidor establecerá el nombre del usuario autenticado en la variable ***USER_AUTH*** (*IIS*) o ***REMOTE_USER*** (*Apache*), accesible mediante el *array* `$_SERVER`.

Sin embargo podemos configurar manualmente el acceso con autenticación en *PHP*. Para ello se debe enviar una cabecera de tipo ***WWW-Authenticate*** mediante la función `header()` (explicada más abajo). Esto enviará dicha cabecera inmediatamente, pausando la ejecución del *script*, causando que se abra una ventana de usuario/contraseña en el navegador. Si la autenticación es correcta, se volverá a cargar la misma *URL*, pero esta vez las variables del servidor ***PHP_AUTH_USER***, ***PHP_AUTH_PW*** y ***AUTH_TYPE*** establecidas a un valor apropiado (nombre de usuario proporcionado, contraseña y tipo de autenticación, *basic* o *digest*).

Esta acción es exactamente la misma que realiza el servidor automáticamente, si está configurado para hacerlo. La diferencia es que en el caso automático, el servidor se encarga de verificar la validez de las credenciales presentadas y de permitir o no el acceso a la página.

> Las variables ***PHP_AUTH_USER***, ***PHP_AUTH_PW*** y ***AUTH_TYPE*** también reciben el valor adecuado cuando la autenticación se realiza automáticamente a través del servidor.

> Antiguamente existía la directiva *PHP* ***safe_mode***, la cual, si estaba activada, causaba que las variables ***PHP_AUTH_USER*** y ***PHP_AUTH_PW*** no recibían valor alguno cuando la autenticación era automática a través del servidor. Este modo fue eliminado en *PHP* 4.5.

En caso de que la autenticación sea cancelada por el usuario, la ejecución proseguirá desde el punto en que se había pausado.

```php
if (!isset($_SERVER['PHP_AUTH_USER'])) {
    header('WWW-Authenticate: Basic realm="My Realm"');
    header('HTTP/1.0 401 Unauthorized');
    echo 'Texto a enviar si usuario presiona Cancelar';
    exit;
}
else
{
    echo "<p>Hola, {$_SERVER['PHP_AUTH_USER']}.</p>";
    echo "<p>Tu contraseña: {$_SERVER['PHP_AUTH_PW']}.</p>";
}
```

Una vez obtenemos ***PHP_AUTH_USER*** y ***PHP_AUTH_PW***, se podrían chequear en una base de datos de usuarios. Si las credenciales son correctas se realizará una acción, y si no, otra.

Los tipos de autenticación son *basic* (no se encriptan los datos antes de enviarse) o *digest* (datos se encriptan antes de enviarse). De todas formas, con protocolo *HTTPS*, los datos viajarán encriptados, aunque usemos el modo *basic*. Por ello, el método *basic* sobre conexiones no *HTTPS* está desaconsejado completamente.

Por otro lado, al autenticarse, se debe definir un *realm*, que será un *string* (arbitrario) compartido por varias páginas. Solo hay que autenticarse en una de las páginas de un mismo *realm* para poder visitarlas todas (dentro del mismo sitio *web*).

### header()

Esta función envía una cabecera al cliente. Se debe invocar **antes** de cualquier salida (*HTML*, `echo` o similar, o incluso espacio en blanco).

El primer parámetro es un *string* que define el tipo de cabecera y su contenido, del tipo ***Tipo: valor***.

Existen dos casos especiales:

- Si empieza con ***HTTP/*** se trata del código de estado de respuesta *HTTP*.
- Una cabecera de tipo ***Location*** causará una redirección.

El segundo parámetro (opcional) define si una cabecera debe reemplazar una cabecera anterior del mismo tipo. Por defecto sí lo hace, pero dándole valor ***false*** puede haber varias cabeceras del mismo tipo.

Un tercer parámetro, opcional también, indica un código de respuesta (siempre que el encabezado no esté vacío).

## Cookies

Se pueden crear *cookies* con *PHP*, que son enviadas al navegador y almacenadas localmente. A su vez, las *cookies* vigentes se envían al servidor a cada *request*. Al igual que los encabezados, es importante enviar las *cookies* estrictamente antes de que se produzca algún tipo de salida.

`setcookie()` crea una *cookie* con el nombre definido en el primer argumento y el valor (opcional, por defecto un *string* vacío) en el segundo. Entre los parámetros que toma la función podríamos destacar el tercero (opcional), que indica la caducidad como un *timestamp*, normalmente el valor de `time()` más los segundos que deseemos. Si tiene éxito retorna ***true*** (no implica que el usuario la haya aceptado), de lo contrario (por ejemplo si hay salida anterior a su envío) ***false***.

Los valores de las *cookies* son accesibles a través del *array* ***\$\_COOKIE***.

Se puede crear una *array cookie*:

```php
setcookie('galleta[uno]', 'Contenido de la uno');
setcookie('galleta[dos]', 'Contenido de la dos');
```

En el ejemplo, `$_COOKIE['galleta']` sería un *array* con dos elementos.

Los valores son codificados para los navegadores (*URL encoded*). Si no queremos esto, se debe usar `setrawcookie()`.

Para eliminar una *cookie* se debe crear con una caducidad anterior al momento actual:

```php
setcookie('galleta', '', 1);  // caducada
```

## Sesiones

Permiten persistir datos entre *requests*. A un usuario se le asigna una ID de sesión, que se almacena en una *cookie* o bien se incluye en la *URL*. Los datos de la sesión son accesibles a través del *array* ***\$\_SESSION***. Para crear nuevos datos de sesión también se hace simplemente a través de esta variable.

Invocando `session_start()`, *PHP* comprueba si se ha recibido una *session ID*, y en caso afirmativo, recreará los valores de esa sesión. Si al llamar a `session_start()` no existe una *session ID*, se crea una nueva ID y se inicia una sesión nueva.

Se puede configurar *PHP* para que esa comprobación sea automática, pero por defecto no lo es. Por otro lado, por defecto, los datos de una sesión se almacenan en un archivo llano en el servidor. Este comportamiento es configurable.

Al terminar el *script PHP* se cierra la sesión, guardándose los datos de la misma para una futura *request*. De puede forzar este cierre y guardado mediante `session_write_close()`.

Para eliminar un dato de sesión, se procede como una variable corriente:

```php
unset($_SESSION['ciudad']);
```

Para eliminar una sesión completamente se usa `session_destroy()`.

## Terminar la ejecución

La sentencia `exit` y su equivalente `die` terminan la ejecución del *script*. No son funciones, sino construcciones del lenguaje. Se pueden escribir sin paréntesis, a no ser que deseemos pasarles un argumento. Este puede ser un *string*, en cuyo caso se enviará este a la salida, o un entero entre 0 (salida correcta) y 254.
