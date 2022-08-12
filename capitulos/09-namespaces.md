# Namespaces

Los *namespaces* sirven para encapsular clases, interfaces, funciones y constantes (no variables). Los nombres de los *namespaces* son *case-insensitive*.

Un archivo que define un *namespace* debe declararlo con `namespace` al principio del archivo, antes de cualquier código (a excepción de `declare`, u otro *namespace*), incluyendo otro código *no-PHP*.

Simplemente de declara:

```php
namespace Nombre;
```

Definirá un *namespace*, representando una ruta absoluta de *namespaces* desde el *namespace* global (no se puede definir un nombre *fully qualified*, es decir que empiece por ***'\\'***). Cuando se incluye un archivo, aunque lo incluyamos desde dentro de un *namespace*, el archivo incluido está en espacio global si no define `namespace`, o si lo define, estará en el espacio definido.

Una sintaxis alternativa es:

```php
namespace Nombre
{
    /* ... */
}
```

En esta sintaxis, **todo** el código debe ir entre llaves (e excepción de una sentencia `declare` al principio).

## Subnamespaces

Es posible crear una estructura jerárquica de *namespaces*.:

```php
namespace MiProyecto\Niveles\Nivel0;
```

## Múltiples *namespaces* en un archivo

Un archivo puede contener múltiples *namespaces*, aunque no está recomendado hacerlo:


```php
namespace Nombre1;
/* Código en el namespace Nombre1 */
namespace Nombre2;
/* Código en el namespace Nombre2 */
```

Si queremos combinar código perteneciente a un *namespace* con código sin *namespace*, solo se puede hacer con la sintaxis entre llaves, y el código sin *namespace* debe ir dentro de un bloque `namespace` sin nombre:

```php
namespace Nombre
{
    /* Código en el namespace Nombre */
}

namespace
{
    /* Código sin namespace */
}
```

## Uso de los *namespaces*

Cuando hacemos referencia o llamada a un elemento ***foo*** (exceptuando variables) sin cualificar (sin especificar *namespace*), se resuelve como ***\<currentNS\>\\foo***, donde ***\<currentNS\>*** es el *namespace* desde el que se realiza la llamada.

Si hacemos la referencia cualificada relativa, del tipo ***\<NS\>\\foo***, se resolverá como ***\<currentNS\>\\\<NS\>\\foo***. Si lo hacemos con cualificación absoluta, del tipo del tipo ***\\\<NS\>\\foo***, se resolverá efectivamente como ***\\\<NS\>\\foo***.

Para referirnos explícitamente a elementos del *namespace* global, simplemente se deberá incluir un carácter ***\\*** delante del identificador (`$a = new \Clase;`).

## Uso de mecanismos dinámicos

Cuando utilizamos variables que contienen nombres de funciones o clases para invocar o instanciar, debemos indicar el nombre *fully qualified*. Esto se aplica también, por ejemplo, a la definición de constantes con `define`.

```php
namespace MiEspacio;    // entramos en el espacio MiEspacio
class MiClase { /* ... */ }
$a = new MiClase;    // OK, estamos en el mismo espacio que
                     // la definición de la clase
$b = new \MiEspacio\MiClase;    // OK, equivale al anterior
$nombre = 'MiClase';
$c = new $nombre;  // Error, a través de instanciación
                   // dinámica hay que especificar namespace
$nombre = '\MiEspacio\MiClase';
$d = new $nombre;    // OK, ahora sí
```

Para construir *strings* con nombre qualified, se puede usar la constante `__NAMESPACE__`, que almacena la ruta completa del *namespace* actual.

Por otro lado, la palabra clave `namespace`, a parte de servir para declarar un *namespace*, también sirve para indicar el *namespace actual*:

```php
namespace\foo();    // equivale simplemente a: foo();
namespace\sub\foo();    // equivale a: sub\foo();
```

En los ejemplos, `namespace` se sustituye por el nombre *fully qualified* del *namespace* actual.

## Importación

Importar es la posibilidad de referirnos a un objeto externo mediante un alias, sin cualificar. Es posible crear alias para elementos de un *namespace*, así como de un *namespace* en sí. Este *aliasing* se realiza con el operador `use`... `as`.

Las importaciones se realizan relativas al espacio global, no al actual, con lo que no es necesario (y no se recomienda) empezar el nombre del espacio a importar con barra invertida (***\\***).

Si importamos una función o constante, hay que informar en el código de que se trata de una función (`use function`) o constante (`use const`), no así cuando importamos un *namespace* o una clase.

Cuando omitimos `as`, se entenderá que este será el último elemento de la importación: `use <un>\<elemento>\<a>\<importar>` equivale a `use <un>\<elemento>\<a>\<importar> as <importar>`.

```php
namespace MiEspacio;  // entramos en el namespace MiEspacio
use Un\Espacio\Exterior as OtroEspacio;
use Un\Espacio\Lejano;
use UnaClase;    // importamos una clase en namespace global
use function Un\Espacio\Exterior\foo;
use const Un\Espacio\Exterior\CONSTANTE;

$a = new OtroEspacio\Clase;    // instancia de
                               // Un\Espacio\Exterior\Clase
$b = new Lejano\Clase;    // instancia de
                          // Un\Espacio\Lejano\Clase
$c = new UnaClase;    // instancia de \UnaClase, y no de
                      // \MiEspacio\UnaClase
foo();    // llamada a Un\Espacio\Exterior\foo()
echo CONSTANTE;    // muestra Un\Espacio\Exterior\CONSTANTE
```

Se pueden combinar varios `use` en una misma línea, separados por coma (***,***):

```php
use Un\Espacio\Exterior as OtroEspacio,
    \Un\Espacio\Lejano as MasEspacio,
    \Un\Espacio\Adicional;
```

Se pueden usar los nombres importados como valores para variables para usos dinámicos.

Los nombres *fully qualified* en nuestro código no se ven afectados por las importaciones.

Las importaciones no se pueden hacer dentro de un bloque de código, y afectan al archivo en el que están definidas, por lo que no afectarán a los archivos que incluyamos o los que nos hayan incluido.

Si deseamos incluir varios elementos de un mismo *namespace*, los podemos agrupar:

```php
use Un\Espacio\Exterior\{ClaseA, ClaseB as B, ClaseC};
use function Un\Espacio\Exterior\{fooA, fooB, fooC as theC};
use const Un\Espacio\Exterior\{A, B as CONSTB, C};
```

## Fallback a función/constante global

Las referencias no cualificadas a nombres de clases dentro de un *namespace* se resuelven al *namespace* actual. Si no encuentra ese nombre definido en el espacio actual, no buscará más. En cambio, las referencias *unqualified* de constantes y funciones que no se encuentren definidas en el espacio local, se buscarán en el espacio global.

## Reglas de resolución de nombres

Al hacer referencia a un nombre dentro del código, los nombres se resuelven así:

- Los nombres *fully qualified* se resuelven sin el separador inicial: ***\\A\\B\\C*** se resuelve a ***A\\B\\C***.
- Los nombres relativos (usando `namespace`) se resuelven sustituyendo ***namespace*** por el espacio actual: ***namespace\\A\\B*** dentro del espacio ***X\\Y*** se resuelve a ***X\\Y\\A\\B***, y dentro del espacio global se resuelve a ***A\\B***.
- Los nombres cualificados (***A\\B\\C***) son sustituidos según importación definida, y si no procede esta, se añade al principio el nombre del espacio actual.
- Los nombres sin cualificar (***A***) son sustituidos según importación definida, y si esta no procede, pueden pasar dos cosas:
    - Si es el nombre de una clase, se le prefija el nombre del *namespace* actual.
    - Si es una función o constante, también se prefija el nombre del espacio actual, pero en este caso, si no se encuentra, pero sí está disponible en el espacio global, en *runtime* usará la versión del *namespace* global.
