# KnpMenu

Esta librería facilita la creación de menús.

## Instalación

Desde el directorio de nuestro proyecto, usaremos `composer`:

```
composer require knplabs/knp-menu
```
La librería se instala en ***vendor/knplabs/knp-menu***. Nuestra aplicación debe incluir el *autoload* de los paquetes de *composer*:

```
require __DIR__ . '/vendor/autoload.php';
```

El *namespace* base del paquete es ***Knp\\Menu***, y se corresponde con el directorio ***vendor/knplabs/knp-menu/src/Knp/Menu***.

## Bases

Veamos el uso básico a partir de un ejemplo:

```php
use Knp\Menu\Matcher\Matcher;
use Knp\Menu\MenuFactory;
use Knp\Menu\Renderer\ListRenderer;

$factory = new MenuFactory();
$menu = $factory->createItem('My menu');
$menu->addChild('Home', ['uri' => '/']);
$menu->addChild('Comments', ['uri' => '#comments']);
$menu->addChild('Symfony', [
    'uri' => 'http://symfony.com/'
]);

$renderer = new ListRenderer(new Matcher());
echo $renderer->render($menu);
```

En primer lugar creamos una instancia de una factoría ***Knp\\Menu\\MenuFactory*** (***\$factory***) que nos servirá para crear los menús. Al invocar el método `createItem()` de esta factoría, pasándole un nombre para nuestro menú, obtenemos una instancia del nuevo menú (***\$menu***).

Para añadir elementos a nuestro menú usaremos su método `addChild()`, al que le pasaremos, por un lado, un nombre, y por otro un *array* de opciones:

- ***uri***: indicará la *URI* de destino. También se puede indicar una *URL*. Si no se indica, la opción de menú será un simple texto sin enlace.
- ***label***: texto del elemento (si no existe, usará su nombre).
- ***attributes***: *array* con los atributos deseados para la entrada de menú. Si el menú se renderiza como una lista, definirá los atributos *HTML* de la etiqueta `<li>`.

Una vez configurada la instancia del menú deseado, se debe renderizar. Esto se hace mediante una instancia del renderizador que deseemos (en el ejemplo, ***Knp\\Menu\\Renderer\\ListRenderer***, que es el habitual). Este renderizador necesita recibir una instancia de un objeto *matcher*, que se usa para tener constancia de cuál es el elemento actual (*current*) del menú (se verá más adelante).

Usando el mencionado renderizador, el ejemplo anterior produce una etiqueta `<ul>` en la que anida tres etiquetas `<li>`, con sus correspondientes textos y enlaces. Los elementos `<li>` pueden anidarse. En cada nivel, el primer y último elemento obtendrán automáticamente la clase ***first*** y ***last*** respectivamente. El elemento de menú correspondiente a la página actual recibirá, además, la clase ***current***, mientras que sus *ancestors* recibirán la clase ***current_ancestor***.

El renderizado se realizará en un código *HTML* debidamente espaciado e indentado para facilitar su inspección y depuración. Si no nos interesa tal formato, se puede pasar como segundo parámetro al constructor del renderizador un *array* que contenga la clave ***compressed*** puesta a ***true***:

```php
$renderer = new ListRenderer(new Matcher,
                             ['compressed' => true]);
```

Cada vez que se utilice este renderizador, se producirá un menú con *HTML* condensado (lo cual no afecta a su visualización). Si en lugar de ello queremos que solo algunos menús (en caso de disponer de varios) se rendericen con *HTML* condensado, se construirá el renderizador sin esa opción, y se indicará la misma en el momento de renderizar un menú con el método `render()` del renderizador:

```php
echo $renderer->render($menu, ['compressed' => true]);
```

Para desarrollar un renderizador, se puede extender la clase ***ListRenderer***, o hacer un renderizador nuevo que implemente la interfaz ***Knp\\Menu\\Renderer\\RendererInterface*** (se puede extender también el renderizador genérico ***Knp\\Menu\\Renderer\\Renderer***).

## Árbol de menús

Así como a una instancia de un menú se le pueden añadir elementos mediante `addChild()`, a cada uno de estos elementos se le puede añadir, a su vez, elementos, también mediante el método `addChild()`. De este modo se forma un árbol de menús.

Para acceder a un elemento se usará como clave el nombre definido para dicho elemento.

```php
$menu['Comments']->addChild('My Comments');
$menu['Comments']->addChild('Your Comments');
```

Al renderizar un menú como lista, los elementos hijos se englobarán en una etiqueta `<ul>` que los contendrá (como elementos `<li>`).

Los elementos `<ul>` (a excepción del elemento global) reciben la clase ***menu_level_N***, donde ***N*** es el número de nivel del submenú.

## Atributos de los elementos de menú

El texto con el que aparecerá cada elemento coincidirá por defecto con el nombre dado. Podemos *override* este texto mediante el método `setLabel()` del menú (o con la opción ***label*** del *array* de inicialización).

```php
$menu['Comments']['Your Comments']
    ->setLabel('Tus comentarios');
```

Para establecer la *URI* después de inicializar, se usa `setUri()`, pasándole dicha *URI* o *URL*.

Para establecer el valor de un atributo usaremos `setAttribute()`, pasándole el nombre y el valor de dicho atributo:

```php
$menu['Comments']->setAttribute('id', 'CommentsID');
```

Para eliminar un atributo existente, se establecerá su valor a ***null***.

Se pueden dar attributos al enlace (`<a>`) o etiqueta (`<span>`) de un elemento de menú mediante `setLinkAttribute()` o `setLabelAttribute()` respectivamente.

También es posible dar un atributo a todos los hijos de una opción padre, mediante `setChildrenAttribute()`. Esto dará el atributo a la etiqueta `<ul>` que engloba a los hijos.

```php
$menu->setChildrenAttribute('class',
                            'una-clase');  // elementos de
                                           // máximo nivel
$menu['Comments']->setChildrenAttribute('class',
                                        'otra-clase');
```

## Visibilidad de los elementos

Es posible renderizar solamente los niveles más altos de un menú, usando la opción ***depth***:

```php
$renderer->render($menu, ['depth' => 2]);
```

Por otro lado, se puede activar o desactivar la visibilidad de un elemento de menú y/o de sus hijos. Mediante un booleano pasado a `setDisplay()` se controla la visibilidad del menú y sus hijos, mientras que con `setDisplayChildren()` se controla la visibilidad de los hijos únicamente.

```php
$menu['Comments']->setDisplay(false);
$menu['Home']->setDisplayChildren(false);
```

## El elemento actual

Normalmente, el elemento de menú correspondiente a la página actual deberá aparecer de forma distinta. Para ello, se utiliza un objeto *matcher*, que si encuentra coincidencia entre la página y el elemento `<li>` lo marcará con la clase ***current***. Por otro lado también marcará a los elementos `<li>` *ancestors* del actual con la clase ***current_ancestor***.

Al renderizador hay que indicarle cómo debe hacer el *match* entre la página actual y los elementos de menú. Para ello se le pasará, al construirse, una instancia de un objeto ***Knp\\Menu\\Matcher\\Matcher*** que tiene esa información. Este *matcher* decidirá en base a los objetos *voter* que tenga registrados. El *voter* más usual es el que comprueba simplemente a base de la *URI* actual: ***Knp\\Menu\\Matcher\\Voter\\UriVoter***.

Primero hay que construir ese *voter*, pasándole el *string* que deberá comparar con los campos *URI* de los distintos elementos del menú. Luego hay que asociar ese *voter* al objeto *matcher*. Existen *voters* para comparar también con una expresión regular (***RegexVoter***) o un nombre de ruta (***RouteVoter***). También podemos crear un *voter* nosotros mismos. Para especificar un *voter*, hay que pasar una instancia del mismo al constructor del *matcher*:

```php
use Knp\Menu\Matcher\Matcher;
use Knp\Menu\Matcher\Voter\UriVoter;
use Knp\Menu\Renderer\ListRenderer;

$matcher = new Matcher(
    new UriVoter($_SERVER['REQUEST_URI'])
);
$renderer = new ListRenderer($matcher);
```

Para crear un *voter* debemos crear una clase que implemente la interfaz ***Knp\\Menu\\Matcher\\Voter\\VoterInterface***. Además podemos crear nuestro propio *matcher* (debe implementar ***Knp\\Menu\\Matcher\\MatcherInterface***).

## Iteradores

Para iterar sobre todos los elementos del menú, podemos hacerlo teniendo en cuenta que se trata de una estructura de árbol. Para facilitar el trabajo, podemos recorrer el árbol usando iteradores.

En primer lugar, el iterador ***Knp\\Menu\\Iterator\\RecursiveItemIterator***, al pasarle un objeto menú genera un iterador que va retornando cada uno de los submenús principales, es decir, los hijos del objeto menú.

Para obtener otro iterador que recorra cada uno de los elementos de estos submenús, hay que envolver (*wrap*) el anterior iterador en el iterador *PHP* ***RecursiveIteratorIterator***:

```php
$itemIterator = new
    \Knp\Menu\Iterator\RecursiveItemIterator($menu);
$iterator = new
    \RecursiveIteratorIterator($itemIterator,
        \RecursiveIteratorIterator::SELF_FIRST);
foreach ($iterator as $item) {
    echo $item->getName() . " ";
}
```

La constante ***SELF_FIRST*** del iterador ***RecursiveIteratorIterator*** retorna cada nodo antes que sus hijos (preorden); para hacerlo al revés (postorden), se usa ***CHILD_FIRST***.

El ejemplo no incluye el elemento raíz (el objeto menú en sí). Para que lo incluya se debe pasar a ***RecursiveItemIterator*** un iterador que inicie en tal raíz en lugar del objeto menú:

```php
$rootIterator = new
    \ArrayIterator([$menu]);  // iterador que solo contiene
                              // el nodo raíz
$itemIterator = new
    \Knp\Menu\Iterator\RecursiveItemIterator($rootIterator);
$iterator = new
    \RecursiveIteratorIterator($itemIterator,
        \RecursiveIteratorIterator::SELF_FIRST);
foreach ($iterator as $item) {
    echo $item->getName() . " ";
}
```

## Acceso a las propiedades

Para acceder a las propiedades de los elementos de menú, se dispone de los métodos `getName()`, `getLabel()`, `getUri()`, `getAttribute()`, etc.
