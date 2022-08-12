# Goutte

Esta librería es un *web crawler* (o *web scraper*). Obtiene información de una página *web*, a la que accede a través de un cliente de navegación.

La librería tiene como dependencias varios componentes de *Symfony*, que son: *BrowserKit*, *CssSelector*, *DomCrawler* y *HttpClient*, que proporcionan un cliente (navegador), facilitan el acceso a los elementos del *DOM*, etc. En todo caso, *composer* ya se encarga de mantener las dependencias adecuadamente.

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

El cliente es de tipo ***Goutte\\Client***, y es en realidad una clase que simplemente extiende ***Symfony\\Component\\BrowserKit\\HttpBrowser***, sin añadirle nada más. En versiones antiguas, sin embargo, era una extensión de otro cliente (*Guzzle*). De todas formas, es posible usar cualquiera de los dos clientes, ya que la mayor parte de la funcionalidad es compatible.

Los parámetros del constructor (opcionales todos) son, en este orden:

- Un *array* de parámetros del servidor, equivalente a ***\$\_SERVER***. Por defecto, *array* vacío.
- Un objeto historial (***Symfony\\Component\\BrowserKit\\History***). Por defecto ***NULL***.
- Un objeto *cookie jar* (***Symfony\\Component\\BrowserKit\\CookieJar***). Por defecto ***NULL***.

Una vez tenemos el cliente, podemos hacer una petición mediante un método como `request()`. Este método cambiará el estado del cliente (cambia la página que mostraría el navegador) y retorna un objeto *crawler* (***Symfony\\Component\\DomCrawler\\Crawler***), que es la respuesta recibida, de la que podremos extraer información posteriormente:

```php
$crawler = $cliente->request('GET', 'https://google.es');
```

El primer argumento a `request()` el el método *HTTP*, y el segundo es la *URI* o *URL*. El resto de argumentos son opcionales, de los que destacaremos el primero de ellos, consistente en un *array* de parámetros, del tipo `['clave' => 'valor',...]`.

El método `jsonRequest()`, con los mismos parámetros, convierte el *array* de parámetros a *JSON* y establece las cabeceras adecuadas (***CONTENT_TYPE*** y ***HTTP_ACCEPT***).

Por otro lado, el método `xmlHttpRequest()`, con los mismos parámetros también, se usa para peticiones *AJAX*. Establece automáticamente la cabecera ***HTTP_X_REQUESTED_WITH***.

### Posible problema SSL

Al intentar acceder a una página mediante protocolo *HTTPS*, puede aparecer un mensaje similar al siguiente:

> fopen(): SSL operation failed with code 1. OpenSSL Error messages: error:1416F086:SSL routines:tls_process_server_certificate:certificate verify failed

Este error se debe a que *PHP* no localiza los certificados necesarios para el protocolo *SSL*. Para arreglarlo, se debe ir a la página de `curl` (https://curl.se) y en el apartado de documentación buscar el *CA Bundle* (https://curl.se/docs/caextract.html). Descargar archivo ***cacert.pem*** y guardarlo en algún directorio del servidor. Tras esto, editar ***php.ini***, variable ***curl.cainfo*** (sección ***[curl]***), dándole el *full path* de ese archivo descargado. Tras reiniciar el servidor *web*, debería funcionar bien.

Por ejemplo, la línea de *php.ini* podría quedar, en un servidor *WAMP*:

```
curl.cainfo = "c:\wamp64\bin\php\cacert.pem"
```

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

El tercer argumento permite indicar el método *HTTP*, mientras el cuarto es un *array* que permite cambiar el valor de los valores ***\$\_SERVER***, como por ejemplo el valor de una cabecera.

```php
$cliente->submitForm('Log in', ['login' => 'my_user',
                     'password' => 'my_pass'],
                     'PUT',['HTTP_ACCEPT_LANGUAGE' => 'es']
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

## Cookies

Para obtener todas las *cookies* usaremos el método del cliente `getCookieJar()`. El objeto *cookie* posee numerosos métodos para obtener sus datos:

```php
$cookieJar = $cliente->getCookieJar();
$cookie = $cookieJar->get('nombre_cookie');  // obtiene la
                                             // cookie

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

Para crear *cookies* podemos trabajar con objetos de la clase ***Cookie*** y ***CookieJar***, ambos pertenecientes al *namespace* ***Symfony\\Component\\BrowserKit***.

```php
use Goutte\Client;
use Symfony\Component\BrowserKit\Cookie;
use Symfony\Component\BrowserKit\CookieJar;

$cookie = new Cookie('nombre', 'valor',
                     strtotime('+1 day'));
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

## El objeto *crawler*

El objeto *crawler* (***Symfony\\Component\\DomCrawler\\Crawler***) contiene un objeto construido a partir de la respuesta recibida del servidor (normalmente, el archivo *HTML* recibido). Se puede crear un *crawler* pasándole simplemente un *string* con dicho código *HTML*:

```php
use Symfony\Component\DomCrawler\Crawler;

$crawler = new Crawler($html);
```

El *crawler* se compone de elementos *DOM* (de tipo estándar de *PHP* ***\\DOMElement***), y se puede acceder a ellos mediante una sentencia `foreach` (no se puede tratar como un *array*):

```php
foreach($crawler as $domElement) {
    var_dump($domElement->nodeName);
}
```

Si queremos buscar un elemento concreto dentro de la jerarquía de elementos, existen dos formas de hacerlo: utilizando cualquiera de los métodos del *crawler* `filterXPath()` o `filter()`. Al primero se le debe pasar un *XPath* (se omite la explicación), y al segundo un selector tipo *CSS* con el mismo formato que los selectores de *jQuery*.

```php
$crawler = $crawler->filter('body > p');
```

Dado que los métodos que filtran objetos *crawler* retornan a su vez un objeto *crawler*, se pueden encadenar varios de ellos.

Cuando recibimos el *crawler* tras una petición del cliente, este contiene solamente los elementos del nivel más alto. En una página *web* normal, significa un solo elemento (la etiqueta ***html*** con todo su contenido). Tras aplicar un filtro, contendrá todos los elementos coincidentes del *DOM*.

### Filtrar con selectores CSS

El método `filter()` acepta un *string* con los tipos de selector que indicamos a continuación:

- `*` (asterisco) selecciona todos los elementos (todas las etiquetas).
- `parent > child` selecciona los elementos que coincidan con el selector ***child*** que son descendientes inmediatos de un elemento que coincide con el selector ***parent***. Se puede indicar una sucesión de dos o más elementos.
- `ancestro descendiente` selecciona los elementos coincidentes con ***descendiente*** que son descendientes (inmediatos o no) de un elemento ***ancestro***. Se puede indicar una sucesión de dos o más elementos.
- `.clase` selecciona los elementos con la clase ***clase***.
- `#id` selecciona los elementos con *ID* ***id***.
- `etiqueta` selecciona todos los elementos con la etiqueta *HTML* indicada.
- `prev + next` selecciona los elementos coincidentes con el selector ***next*** que son inmediatamente precedidos por un elemento coincidente con ***prev*** dentro del mismo nivel.
- `prev ~ siblings` selecciona todos los elementos que coincidan con ***siblings*** y vayan después del elemento coincidente con ***prev*** (todos estos elementos deben tener el mismo padre).

También existen los selectores basados en atributos. Consisten en una expresión entre corchetes (***[]***). La expresión compara un atributo del elemento con un valor específico. El valor puede indicarse entrecomillado o no (aquí se indicará entre comillas):

- `[atributo|="valor"]` selecciona solo los elementos cuyo atributo indicado es igual a ***valor***, o empieza por ***valor-*** (nótese el guión).
- `[atributo*="valor"]` solo los elementos cuyo atributo indicado contiene ***valor***.
- `[atributo~="valor"]` elementos cuyo atributo indicado contiene la palabra ***valor*** (delimitada por espacios).
- `[atributo$="valor"]` elementos cuyo atributo indicado termina **exactamente** (*case sensitive*) en ***valor***.
- `[atributo="valor"]` elementos cuyo atributo indicado es igual a ***valor***.
- `[atributo!="valor"]` elementos cuyo atributo indicado no existe o es distinto a ***valor***.
- `[atributo^="valor"]` elementos cuyo atributo indicado empieza por ***valor***.
- `[atributo]` elementos con el atributo indicado (con cualquier valor).

El otro tipo de selector consiste en un carácter dos puntos (***:***) seguido de una propiedad concreta:

- `:animated` solo elementos que estén en proceso de animación en ese momento.
- `:button` solo botones.
- `:checkbox` solo *checkboxes*.
- `:checked` solo elementos marcados o seleccionados.
- `:contains()` selecciona únicamente los elementos que contengan el texto especificado, el cual se puede indicar entrecomillado o no.
- `:disabled` elementos deshabilitados.
- `:empty` selecciona elementos sin descendientes (inculyendo nodos de texto).
- `:enabled` elementos habilitados.
- `:first-child` selecciona los elementos que sean el primer hijo de su padre.
- `:last-child` selecciona los elementos que sean el último hijo de su padre.
- `:first-of-type` selecciona los elementos del tipo especificado que sean el primero de ese tipo entre sus posibles hermanos.
- `:last-of-type` selecciona los elementos del tipo especificado que sean el último de ese tipo entre sus posibles hermanos.
- `:lang()` selecciona los elementos con el idioma indicado (***es***, ***en***, etc.). El indicador de idioma puede ir entrecomillado o no.
- `:nth-child()` selecciona los elementos que sean el n-ésimo hijo de su padre (1 corresponde al primero).
- `:nth-last-child()` selecciona los elementos que sean el n-ésimo hijo de su padre por la cola (1 corresponde al último).
- `:nth-of-type()` selecciona los elementos del tipo especificado que sean el enésimo de ese tipo entre sus posibles hermanos.
- `:nth-last-of-type()` selecciona los elementos del tipo especificado que sean el enésimo de ese tipo por la cola entre sus posibles hermanos.
- `:only-child` selecciona los elementos que son los únicos hijos de su padre.
- `:only-of-type` selecciona los elementos que son los únicos hijos de su tipo tiene su padre.
- `:root` selecciona los elementos del nivel raíz.
- `:selected` selecciona los elementos seleccionados.

Los diversos tipos de selectores pueden juntarse y combinarse en varios componentes cuando tenga sentido. En ese caso, deberán cumplirse todos ellos. Si queremos combinar (sumar) los resultados de varios selectores, los separaremos con comas: `selector1, selector2, selector3`. Cuando entre dos selectores no hay operador, no debe haber espacio entre ellos, ya que el espacio solo es un operador en sí (`ancestro descendiente`).

Es posible encadenar filtros, y también se pueden fragmentar los diferentes componentes de un selector compuesto. En este sentido, las dos sentencias siguientes son equivalentes:

```php
$crawler1 = $crawler->filter('p.clase[atributo]');
$crawler1 = $crawler->filter('p')->filter('.clase')
    ->filter('[atributo]');
```

### Otros métodos de filtrado

A parte de `filter()`, existen otros métodos usados para refinar los filtros:

- `eq()` recibe un número de índice, de tal modo que solo el elemento en ese índice quedará seleccionado.
- `first()` selecciona únicamente el primero de los elementos encontrados.
- `last()` selecciona únicamente el último de los elementos encontrados.

También se puede filtrar utilizando una función anónima que se pasará al método `reduce()`:

```php
$crawler = $crawler
    ->filter('body > p')
    ->reduce(function(Crawler $nodo, $i) {
        return ($i % 2) == 0;
    });
```

La función anónima va pasando por todos los nodos, y recibe, por un lado, un *crawler* que contiene cada nodo, y por otro, el número de índice (*zero-based*) de cada uno dentro del *crawler* global (que contiene todos los nodos a tratar). Si queremos que el nodo concreto que está tratando la función desaparezca de la selección retornada por el método `reduce()`, retornaremos ***false***.

En las siguientes explicaciones, al referirnos a **selección actual** se entenderá el conjunto de todos los nodos presentes en el *crawler* actualmente, y **nodo actual** al primero de esos nodos.

El método `siblings()` hace que la nueva selección esté formada por todos los nodos hermanos del nodo actual (sin incluir a este). En cambio, `nextAll()` es como `siblings()`, pero solo tiene en cuenta a los hermanos **posteriores** del nodo actual. El método `previousAll()` hace lo mismo con los precedentes.

El método `children()` retorna una lista de nodos correspondientes a todos los hijos **inmediatos** del nodo actual (en el orden en que están definidos), sin incluir el nodo en cuestión. Por otro lado, el método `ancestors()` hace lo propio con todos los ancestros de ese nodo actual, desde el más interno hasta de máximo nivel (*document root*). Si solo deseamos el nodo padre, se usa el método `closest()`.

### Acceso al valor de los nodos

Estos métodos del *crawler* acceden a propiedades del nodo actual. Si el *crawler* no contiene ningún nodo, se producirá un error.

`nodeName()` retorna el nombre del nodo (etiqueta *HTML*).

El método `text()` accede a todo el texto que contiene el nodo (incluyendo el texto de sus descendientes). Si le pasamos un *string*, lo retornará en caso de que el *crawler* esté vacío. Por otro lado, este método aplica *trim* de espacios a los textos. Si se desea que no se aplique tal cosa, se le debe pasar ***false*** como segundo argumento.

El método `attr()` retorna el valor del atributo indicado como primer argumento.

El método `extract()` actúa sobre todos los nodos de la lista, no solo sobre el primero. Sirve para obtener el valor de varios atributos a la vez. Recibe como argumento un *array* con los nombres de los atributos que nos interesan, y retorna un *array*, cada elemento del cual es a su vez un *array* con los valores de los atributos (un elemento para cada nodo). Los atributos especiales ***\_name*** y ***\_text*** representan respectivamente el nombre del nodo (etiqueta *HTML*) y el valor (texto) del mismo.

El método `each()` recibe una función anónima, la cual recibe, al igual que el método `reduce()`, un *crawler* con el nodo a procesar y el número de índice del nodo. Pero en esta ocasión, el método retorna un *array* con los valores que va retornando la función anónima para cada nodo. Estos valores pueden ser de cualquier tipo.

### Añadir contenido

Solo se puede añadir contenido (un *string* con un documento *HTML* o *XML*) una única vez al *crawler*. Existen distintas opciones:

- A través de una *request* al cliente, como se ha visto ya.
- En el constructor del *crawler* (visto también).
- Mediante uno de los siguientes métodos del *crawler*:
    - `addHtmlContent()`
    - `addXmlContent()`
    - `addContent()`
    - `add()`

Estos métodos reciben un *string* con el contenido deseado. `addHtmlContent()` y `addXmlContent()` asumen *UTF-8* por defecto, aunque puede cambiarse si se le pasa como segundo argumento la codificación a usar. El primero está pensado para contenido *HTML*, mientras el segundo se usa para contenido *XML*. Por otro lado, `addContent()` se puede usar por cualquier tipo de contenido, e intenta adivinar el juego de caracteres según el contenido, y si no puede adivinarlo usa ***ISO-8859-1***. `add()` también se puede usar para cualquier tipo de contenido.

### Enlaces

Para seleccionar un enlace, puede hacerse mediante filtros, o usando el método `selectLink()`, que permite seleccionar usando el contenido del enlace (se le pasa dicho contenido en un *string* como argumento).

Una vez el *crawler* contiene como primer elemento el enlace deseado, podemos obtener el objeto enlace (***Symfony\\Component\\DomCrawler\\Link***) mediante el método `link()`:

```php
$crawler = $crawler->selectLink('Aceptar');
$link = $crawler->link();
```

Al método `link()` se le puede pasar opcionalmente el método *HTTL* (por defecto es ***get***).

El objeto enlace posee algunos métodos útiles:

`getUri()` retorna la *URL* asociada al enlace. Aunque el atributo ***href*** sea una *URI* o ruta relativa, se reconstruye y retorna siempre la *URL* completa. En el caso de que el enlace sea relativo, es necesario que el *crawler* posea información sobre al *URL* base. En este caso, si dicho *crawler* se construye a través del constructor, debemos indicarle dicha *URL* base en el tercer argumento del mismo (el primero y segundo son ***NULL*** por defecto). Esta *URL* base es en realidad la *URL* completa de la página:

```php
$crawler = new Crawler(NULL, NULL,
                       'http://dominio.com/pagina.html');
```

Si obtenemos el *crawler* a través de una *request* al cliente, ya se realiza correctamente la inicialización del mismo.

`getNode()` retorna el elemento *DOM* (objeto ***\\DOMElement***) asociado al enlace.

`getMethod()` retorna el método *HTTP* del enlace.

### Imágenes

Es posible seleccionar imágenes a través de su atributo ***alt***, mediante el método del *crawler* `selectImage()`, a la que se le pasa el texto de dicho atributo como primer argumento.

Una vez el *crawler* tiene como primer elemento el nodo correspondiente a la imagen deseada, podemos obtener el objeto imagen (***Symfony\\Component\\DomCrawler\\Image***) correspondiente mediante el método `image()`:

```php
$crawler = $crawler->selectImage('Un gatito');
$imagen = $crawler->image();
```

Un objeto imagen dispone también de métod `getUri()`.

### Formularios

El método `selectButton()` del *crawler* busca elementos de tipo `<button>`, `<input type="submit">`, `<input type="button">`, o elementos `<img>` dentro de estos. El argumento que se le pasa al método se compara con los atributos ***id***, ***alt***, ***name***, y ***value*** de estos elementos, así como el texto que puedan contener.

Una vez tenemos uno de estos elementos como primer elemento de la selección, el método `form()` retorna el objeto formulario (***Symfony\\Component\\DomCrawler\\Form***) que contiene ese elemento. Este objeto formulario tiene varios métodos útiles para obtener información, de los que destacamos:

- `getFormNode()` retorna el elemento *DOM* (objeto ***\\DOMElement***) asociado al formulario.
- `getName()` retorna el nombre del formulario, si está definido (si no, retorna un *string* vacío).
- `getUri()` retorna el enlace asociado a la acción de envío del formulario. Si el método es ***GET***, incluso le añade los parámetros, de tal forma que el enlace imita lo que haría el navegador.
- `getMethod()` retorna el método *HTTP* del formulario. Si no está definido, retorna el *string* ***GET***.
- `has()` recibe el nombre de un campo del formulario, y retorna un booleano según dicho campo exista o no.
- `remove()` recibe el nombre de un campo del formulario, y lo elimina del mismo.

Es posible rellenar los datos deseados del formulario en el momento de obtenerlo, pasando al constructor un *array* (opcional):

```php
$form = $crawler->selectButton('mi-botón')
    ->form([
        'nombre' => 'Ambrosio',
        'apellido' => 'Smith'
    ]);
```

Para establecer el valor de los campos a partir de una instancia de objeto formulario, disponemos del método `setValues()`, al que se le debe pasar un *array* con los nombres de los campos y sus valores. Hay que tener en cuenta que el *array* debe ser plano, no multidimensional, con lo que los parámetros *array* se indicarán como una clave llana:

```php
$form->setValues([
    'nombre' => 'Ambrosio',
    'telefono[casa]' => '5552211',
    'telefono[trabajo]' => '5552233'
]);
```

Para leer los valores establecidos en los campos, se usa `setValues()`, el cual retorna también un *array* llano, aunque existan parámetros *array* (incluso parámetros multidimensionales):

```php
$valores = $form->getValues();
```

En el ejemplo anterior, retornaría el *array*:

```php
[
    'nombre' => 'Ambrosio',
    'telefono[casa]' => '5552211',
    'telefono[trabajo]' => '5552233'
]
```

Sin embargo, podríamos usar el método `getPhpValues()` y obtendríamos u *array* multidimensional cuando procediera:

```php
$valores = $form->getPhpValues();
```

En este caso, retornaría el *array*:

```php
[
    'nombre' => 'Ambrosio',
    'telefono' => [
        'casa' => '5552211',
        'trabajo' => '5552233'
    ]
]
```

De forma similar, podemos obtener los valores de campos de tipo archivo mediante `getFiles()` y `getPhpFiles()`.

Por otro lado, es posible interactuar con el formulario como si lo hiciésemos desde el navegador, accediendo a los campos del formulario usándolo como un *array*, combinado con métodos como `setValue()`, `tick()`, `untick()`, `select()` o `upload()`:

```php
$form['username']->setValue('usuario33');
// Equivalente:
$form['username'] = 'usuario33';

// Check o uncheck un checkbox:
$form['terminos']->tick();
$form['terminos']->untick();

// Seleccionar una opción:
$form['nacimiento[year]']->select(1984);

// Seleccionar varias opciones de un selector múltiple:
$form['intereses']->select(['laravel', 'ganchillo']);

// Simular la subida de un archivo:
$form['foto']->upload('/ruta/absoluta/a/ambrosio.jpg');
```

Como se ha visto, cuando el campo es de tipo *array* se accederá al elemento incorporando el índice en el *string* de clave (`$form['nacimiento[year]']`).

Veamos un ejemplo completo de cómo accederíamos a un formulario de *login*, introduciríamos la información y lo enviaríamos:

```php
use Goutte\Client;

$cliente = new Client();
$crawler = $cliente
    ->request('GET', 'https://github.com/login');

$form = $crawler->selectButton('Sign in')->form();
$form['login'] = 'dev-pep';
$form['password'] = 'password';

$crawler = $cliente->submit($form);
```

#### Seleccionar valores inválidos

Por defecto, los campos de selección múltiple (*radio* y *select*) tienen validación interna para evitar que se seleccionen valores no válidos. Si se desea obviar esta validación y establecer valores inválidos, se puede usar el método `disableValidation()` en todo el formulario o en campos específicos:

```php
// Campo específico:
$form['país']->disableValidation()->select('Invalid value');

// Todo el formulario:
$form->disableValidation();
$form['country']->select('Invalid value');
```

## Resolución de *URIs*

La clase ***Symfony\\Component\\DomCrawler\\UriResolver*** es una clase *helper* que toma una *URI* (relativa, absoluta, fragmento, etc.) y la convierte en una *URL* absoluta contra una *URL* base:

```php
use Symfony\Component\DomCrawler\UriResolver;

// http://localhost/foo:
UriResolver::resolve('/foo', 'http://localhost/bar/foo/');
// http://localhost/bar?a=b:
UriResolver::resolve('?a=b', 'http://localhost/bar#foo');
// http://localhost/:
UriResolver::resolve('../../', 'http://localhost/');
```
