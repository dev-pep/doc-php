# Goutte

Esta librería es un *web crawler* (o *web scraper*). Obtiene información de una página *web*, a la que accede a través de un cliente de navegación.

La librería tiene como dependencias varios componentes de *Symfony*, que son: *BrowserKit*, *CssSelector*, *DomCrawler* y *HttpClient*, que proporcionan un cliente (navegador), facilitan el acceso al *DOM*, etc. En todo caso, *composer* ya se encarga de mantener las dependencias adecuadamente.

## Instalación

Desde el directorio de nuestro proyecto, usaremos `composer`:

```
composer require fabpot/goutte
```

La librería se instala en ***vendor/fabpot/goutte***. Nuestra aplicación debe incluir el *autoload* de los paquetes de *composer*:

```
require __DIR__ . '/vendor/autoload.php';
```

El *namespace* base del paquete es ***Goutte***, y se corresponde con el directorio ***vendor/fabpot/goutte/Goutte***.

## Cliente

El primer paso es crear un cliente para hacer peticiones:

```php
use Goutte\Client;

$cliente = new Client();
```

Los parámetros del constructor (opcionales todos) son, en este orden:

- Un *array* de parámetros del servidor, equivalente a ***\$_SERVER***. Por defecto, *array* vacío.
- Un objeto historial (***Symfony\Component\BrowserKit\History***). Por defecto ***NULL***.
- Un objeto *cookie jar* (***Symfony\Component\BrowserKit\CookieJar***). Por defecto ***NULL***.

Una vez tenemos el cliente, podemos hacer una petición mediante un método como `request()`. Este método cambiará el estado del cliente (cambia la página que mostraría el navegador) y retorna un objeto *crawler* (***Symfony\Component\DomCrawler\Crawler***), que es la respuesta recibida, de la que podremos extraer información posteriormente:

```php
$crawler = $cliente->request('GET', 'https://google.es');
```

El primer argumento a `request()` el el método *HTTP*, y el segundo es la *URI* o *URL*. El resto de argumentos son opcionales, de los que destacaremos el primero de ellos, consistente en un *array* de parámetros, del tipo `['clave' => 'valor',...]`.

El método `jsonRequest()`, con los mismos parámetros, convierte el *array* de parámetros a *JSON* y establece las cabeceras adecuadas (***CONTENT_TYPE*** y ***HTTP_ACCEPT***).

Por otro lado, el método `xmlHttpRequest()`, con los mismos parámetros también, se usa para peticiones *AJAX*. Establece automáticamente la cabecera ***HTTP_X_REQUESTED_WITH***.

## Clicar enlaces

Para simular un clic, se usa el método `clickLink()` del cliente, al que se le pasa el texto del enlace. Como es habitual, cambia el estado del cliente (como si se hubiera hecho clic en el navegador) y retorna un objeto *crawler*:

```php
$crawler = $cliente->clickLink('Ir allí...');
```

Es posible obtener un objeto enlace, del que podremos obtener propiedades, mediante métodos como `getMethod()` o `getUri()`:

```php
$crawler = $cliente->request('GET', '/producto/123');
$link = $crawler->selectLink('Ir allí...')->link();
$client->click($link);
```

## Formularios

Para hacer un *submit* de un formulario, se usa el método `submitForm()` del cliente. El primer argumento será algo que identifique al botón de envío (un `<button>` o un `<input type="submit">`), ya sea el texto, id, valor o nombre. Es el único argumento obligatorio.

Como segundo argumento podemos indicar un *array* que *overrides* los parámetros a enviar (en el caso de archivos se debe indicar la ruta completa).

El tercer argumento permite indicar el método *HTTP*, mientras el cuarto es un *array* que permite cambiar el valor de los valores ***\$_SERVER***, como por ejemplo el valor de una cabecera.

```php
$cliente->submitForm('Log in', ['login' => 'my_user', 'password' => 'my_pass'],
    'PUT', ['HTTP_ACCEPT_LANGUAGE' => 'es']
);
```

Por otro lado, podríamos obtener el objeto correspondiente al formulario, modificarlo a voluntad y enviarlo:

```php
$crawler = $cliente->request('GET', '/login');
$form = $crawler->selectButton('Log in')->form();
$form['login'] = 'my_user';
$form['password'] = 'my_pass';
$crawler = $cliente->submit($form);
```

## *Cookies*

Para obtener todas las *cookies* usaremos el método del cliente `getCookieJar()`. El objeto *cookie* posee numerosos métodos para obtener sus datos:

```php
$cookieJar = $cliente->getCookieJar();
$cookie = $cookieJar->get('nombre_cookie');  // obtiene la cookie

// Leer datos de la cookie:
$name       = $cookie->getName();
$value      = $cookie->getValue();
$rawValue   = $cookie->getRawValue();
$isSecure   = $cookie->isSecure();
$isHttpOnly = $cookie->isHttpOnly();
$isExpired  = $cookie->isExpired();
$expires    = $cookie->getExpiresTime();
$path       = $cookie->getPath();
$domain     = $cookie->getDomain();
$sameSite   = $cookie->getSameSite();
```

Si queremos todas las *cookies* en un *array*:

```php
$valores = $cookieJar->allValues('http://servidor.com');
```

El argumento a `allValues()` es el nombre del *host*, es decir, el método retornará todas las *cookies* correspondientes al *host* (normalmente debería ser el *host* desde el que se ejecuta el *script*).

Entonces ya podemos acceder a los valores de las *cookies* mediante `$valores['nombre_cookie']`.

Para crear *cookies* podemos trabajar con objetos de la clase ***Cookie*** y ***CookieJar***, ambos pertenecientes al *namespace* ***Symfony\Component\BrowserKit***.

```php
use Goutte\Client;
use Symfony\Component\BrowserKit\Cookie;
use Symfony\Component\BrowserKit\CookieJar;

$cookie = new Cookie('nombre', 'valor', strtotime('+1 day'));
$cookieJar = new CookieJar();
$cookieJar->set($cookie);

$cliente = new Client([], NULL, $cookieJar);
```

El nombre y valor de la *cookie* son obligatorios en el constructor. El siguiente argumento (caducidad) es opcional. Existen otros argumentos.

En el ejemplo se ha creado una *cookie*, que se ha añadido a un nuevo *cookie jar*, el cual se usa como *cookie jar* del nuevo cliente.

Para añadir una nueva *cookie* al *cookie jar* de un cliente ya existente:

```php
$cookieJar = $cliente->getCookieJar();
$cookieJar->set($cookie);
```

O directamente:

```php
$cliente->getCookieJar()->set($cookie);
```

## Historial

El cliente tiene asociado un objeto *historial* que va guardando el historial de navegación. Para ira hacia adelante y hacia atrás en el mismo, se usan lo métodos del cliente `forward()` y `back()`.

Para eliminar el historial (y las *cookies* también), el cliente dispone del método `restart()`, que reinicia el cliente.

## Filtrado del objeto *crawler* (respuesta)

El objeto *crawler* contiene la respuesta recibida del servidor. Se puede crear un *crawler* pasándole simplemente un *string* con código *HTML*.

```php
$crawler = new Crawler($html);
```
