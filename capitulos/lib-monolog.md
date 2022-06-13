# Monolog

Esta librería facilita la creación de *logs*.

## Instalación

Desde el directorio de nuestro proyecto, usaremos `composer`:

```
composer require monolog/monolog
```

La librería se instala en ***vendor/monolog/monolog***. Nuestra aplicación debe incluir el *autoload* de los paquetes de *composer*:

```
require __DIR__ . '/vendor/autoload.php';
```

El *namespace* base del paquete es ***Monolog***, y se corresponde con el directorio ***vendor/monolog/monolog/src/Monolog***.

## Uso

Cada instancia de ***Monolog\\Logger*** contiene un canal (nombre) y una pila de *handlers*. Cada entrada enviada al *logger* debe atravesar la pila (de arriba hacia abajo). Un *handler* puede decidir, a parte de escribir el mensaje o no, propagar la entrada al siguiente *handlers* o no. Si un *handler* tiene la propiedad ***$bubble*** en ***false***, no propagará el mensaje. Los *handlers* disponibles se encuentran en ***Monolog\\Handler***.

Los *handlers* se pueden compartir entre varios *loggers*.

Cada *handler* dispone de un *formatter* que da forma al mensaje. Si no se define, tendrá un *default formatter*.  Los *formatters* disponibles se encuentran en ***Monolog\\Formatter***.

### Niveles de *log*

Cada nivel corresponde a una constante definida. Niveles en orden creciente:

- ***DEBUG***: información de depuración.
- ***INFO***: información relevante.
- ***NOTICE***: eventos significativos.
- ***WARNING***: ocurrencias excepcionales que no llegan a ser un error.
- ***ERROR***: errores que deben ser monitorizados.
- ***CRITICAL***: condiciones críticas.
- ***ALERT***: debe tomarse una acción inmediatemente.
- ***EMERGENCY***: sistema inutilizado.

### Uso del *logger*

```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Handler\FirePHPHandler;

$logger = new Logger('mi_logger');
$logger->pushHandler(new StreamHandler(__DIR__ . '/mi_app.log', Logger::DEBUG));
$logger->pushHandler(new FirePHPHandler());

$logger->info('El logger está preparado.');
```

Como se ve en el ejemplo, lo primero es la creación del *logger*. Al crearlo se le debe pasar un nombre, que será útil en caso de disponer de varios *loggers*.

Seguidamente se insertan dos manejadores en el *logger*. En el caso del ***StreamHandler***, se le debe pasar el nombre del archivo y el nivel mínimo de *log* (solo se escribirán en él los mensajes de dicho nivel o superior). El último manejador insertado es el primero por el que pasarán los mensajes.

Finalmente, ya se pueden enviar mensajes al *logger*. En el ejemplo enviamos un *string* con nivel **info**. Sin embargo disponemos de un método para cada nivel de *log*: `debug()`, `info()`, `notice()`, `warning()`, `error()`, `critical()`, `alert()` y `emergency()`.

### Añadir de datos extra

Es posible proporcionar más datos al *logger* pasando como segundo parámetro un *array* con la información deseada.

Existe otra forma de añadir datos: a través de un *processor*. En este caso, estos datos extra serán añadidos automáticamente a todos los registros enviados al *log*. Un *processor* puede ser cualquier *callable*. Este recibe el registro como parámetro, y debe retornarlo, opcionalmente cambiando su clave ***extra***:

```php
$logger->pushProcessor(function ($record) {
    $record['extra']['dummy'] = 'Hello world!';
    return $record;
});
```

En este caso insertamos en el *logger* un *processor*.

Los datos añadidos de la primera forma se incorporan a un *array* (llamado *context*), y los añadidos mediante *processor* se insertan a un segundo *array* (llamado *extra*). Si no añadimos datos extra, estos dos *arrays* llegan vacíos a los *handlers*.

Se pueden registrar varios *processors*, pero en todo caso, todos ellos serán invocados antes que los *handlers*. La invocación de *processors* forma su propia pila, y se invocarán también de arriba hacia abajo.

Es posible registrar un *processor* solamente en un *handler*, en lugar de registrarlo en el *logger*:

```php
$logger = new Logger('mi_logger');
$handler = new StreamHandler(__DIR__ . '/mi_app.log', Logger::DEBUG);
$handler->pushProcessor( /*...*/ );
$logger->pushHandler($handler);
```

En ese caso el *processor* en cuestión solo tendrá efecto en la ejecución del *handler* en el que está registrado (es posible también registrar una pila de *processors* en un *handler*). En todo caso, los *processors* del *logger* se procesan **antes** que los del *handler*.

*Monolog* proporciona una serie de *built-in processors* en ***Monolog\\Processor***.

### Formato de los registros

Es posible registrar un (y solo un) *formatter* en cada *handler*, mediante el método `setFormatter()` del *handler*, al que se le pasa una instancia del *formatter* deseado. Como ejemplo, el *built-in formatter* ***LineFormatter*** genera una línea de texto. Se le puede pasar una plantilla para esa línea, así como un formato de presentación de la fecha:

```php
use Monolog\Logger;
use Monolog\Handler\StreamHandler;
use Monolog\Formatter\LineFormatter;

$formatoFecha = "d/m/Y, h:i";
$formatoSalida = "%datetime% %channel% (%level_name%) > %message% %context% %extra%\n";

$formatter = new LineFormatter($formatoSalida, $formatoFecha);

$handler = new StreamHandler(__DIR__ . '/mi_app.log', Logger::DEBUG);
$handler->setFormatter($formatter);

$logger = new Logger('mi_log');
$logger->pushHandler($handler);

$logger->info('El logger está preparado');
```

En el ejemplo pueden verse los nombres de los campos, y cómo especificarlos:

- ***datetime*** es la fecha/hora del registro.
- ***channel*** es el nombre del canal. Útil en el caso de que varios *loggers* compartan un *handler*.
- ***level*** y ***level\_name*** son el número y el nombre del nivel de *log* del registro, respectivamente.
- ***message*** es el mensaje en sí que registramos.
- ***context*** y ***extra*** son los *arrays* de datos adicionales vistos anteriormente.

Varios *handlers* pueden compartir un mismo *formatter*.

Para una lista completa de los *handlers*, *formatters* y *processors built-in*, véase la documentación del paquete.
