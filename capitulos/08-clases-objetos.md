# Clases y objetos

Para definir una clase:

```php
class <NombreClase>
{
    /* propiedades y métodos */
}
```

La clase puede contener constantes, variables (propiedades) y funciones (métodos).

Los métodos tienen acceso a la variable `$this`, que contiene el objeto instancia de la clase.

Para crear una nueva instancia de la clase: `$objeto = new <NombreClase>();`

Se puede incluso hacer mediante el contenido de una variable:

```php
class MiClase { /* ... */ }
$nombre = 'MiClase';
$objeto = new $nombre();
```

Si no hay que pasar argumentos al constructor de la clase, se pueden omitir los paréntesis: `$objeto = new MiClase;`

Un objeto existente asignado a una variable contendrá la misma instancia, no una copia de la misma. Sin embargo, una variable no es una referencia de la otra. Veamos un ejemplo para entenderlo:

```php
$instancia = new MiClase();
$asignado = $instancia;
$referencia = &$instancia;
```

Las variables no guardan el contenido de un objeto directamente, sino un *handle*, identificador o "apuntador" al mismo.

En nuestro ejemplo, ***\$instancia*** contiene el *handle* o apuntador a la nueva instancia creada. Al asignarse dicha variable a ***\$asignado***, esta última recibe una copia de tal apuntador, con lo que ambas variables tendrán un apuntador (idéntico) a la misma instancia, pero no comparten el mismo apuntador. Sin embargo, ***\$referencia*** sí es una referencia al mismo *handle* que tiene ***\$instancia***. Así, las tres variables hacen referencia al mismo objeto, a través de dos apuntadores. Ahora, si hacemos:

```php
$instancia = null;
```

el contenido de ***\$instancia*** ya no es ese apuntador a la instancia, sino que ha pasado a ser ***null***. Como el contenido de ***\$instancia*** es compartido también por ***\$referencia***, esta última también tendrá contenido ***null***. Sin embargo, ***\$asignado*** seguirá teniendo como contenido un *handle* a la instancia original.

Se dice que los objetos se pasan por referencia cuando se usan como argumentos. Esto no es exactamente así; lo que sucede es que se pasa **el valor** de ese apuntador, que, efectivamente hace referencia al mismo objeto. Veámoslo con un ejemplo:

```php
class MiClase { public $contenido = 0; }

function foo($objeto)
{
    $objeto->contenido++;
    $objeto = null;
}

function bar(&$objeto)
{
    $objeto->contenido++;
    $objeto = null;
}

$ob1 = new MiClase;
echo $ob1->contenido;
foo($ob1);
echo $ob1->contenido;

$ob2 = new MiClase;
echo $ob2->contenido;
bar($ob2);
echo $ob2->contenido;
```

Tanto la función ***foo()*** como la función ***bar()*** cambian el valor del objeto al que hace referencia su parámetro ***\$objeto***. Esa manipulación puede comprobarse tras la ejecución de la función. Al mismo tiempo, ambas **cambian** el valor del argumento recibido en sí, y mientras ese cambio es efectivo en el argumento llamante a ***bar()***, no es efectivo en ***foo()***, ya que el argumento se pasa estrictamente por valor en esta última.

## Propiedades y métodos

Un método puede tener el mismo nombre que una propiedad. Si se debe acceder a uno u a otro dependerá del contexto.

Es posible utilizar las *variable variables* para acceder a métodos y propiedades:

```php
class MiClase {
    private $mipro;
    public function mimet($a, $b) {/*...*/}
    public function foo() {
        $propiedad = 'mipro';
        $metodo = 'mimet';
        $this->$propiedad = 50;  // equivale a $this->mipro;
        $this->$metodo(1, 2);  // equivale a $this->mimet(1, 2);
    }
}

$inst = MiClase;
$metodo = 'mimet';
$inst->$metodo(5, 5);  // equivale a $inst->mimet(1, 2);
```

## extends

Una clase ***D*** puede ser derivada de una clase ***B***:

```php
class D extends B { /* ... */ }
```

Al *override* un método en la derivada, la *signature* (parámetros y sus tipos, y tipo de retorno) del método hijo debía ser, en anteriores versiones de *PHP*, igual que la del padre. Sin embargo, actualmente es suficiente que sean compatibles, y eso significa que la *signature* del hijo debe respetar las **reglas de varianza**, que se verán más adelante. Además, el método hijo no puede eliminar parámetros del padre, o hacer obligatorios los parámetros opcionales del padre (sí puede hacer opcionales los parámetros obligatorios del padre). Puede, en todo caso, añadir parámetros, pero estos deben ser opcionales.

Puede accederse a los métodos *overriden* del padre o propiedades estáticas mediante `parent`, que se refiere a la **clase padre**, mientras que `self` se refiere a la **clase actual**. Se deben usar estos identificadores con el operador `::`.

Renombrar parámetros en las clases hijas no constituye incompatibilidad, pero puede llevar a error si se usan argumentos con nombre.

## ::class

Se usa para obtener el nombre completamente *qualified* de una clase, incluyendo el *namespace*. Se indica a modo de sufijo: `<NombreClase>::class`. Su resolución es en tiempo de compilación.

A partir de *PHP* 8 se puede aplicar también a objetos, y equivale a llamar a `get_class()` sobre el objeto. La resolución es en *run time*.

## Operador nullsafe

Mientras que el acceso a métodos y propiedades se realiza con el operador `->`, a partir de *PHP* 8 se admite el operador *nullsafe* `?->` en cuyo caso si el objeto es *null*, el acceso retornará simplemente *null*.

## Propiedades

Son las variables miembros de una clase. Se declaran mediante `public`, `protected` o `private`, seguido de una declaración de tipo opcional y la declaración normal. Puede incluir inicialización, pero esta debe ser un valor constante.

Las propiedades son accedidas mediante el operador `->` en un objeto instancia. Desde dentro de la clase se accede, mediante `$this -> propiedad` mientras que las propiedades estáticas son accedidas a través de la clase con el operador `::`. Desde dentro de la clase sería `self::$propiedad`. Desde fuera, sería `$instancia::$propiedad`.

Ejemplos de declaraciones de propiedades:

```php
public int $id = 5;    // entero inicializado
private ?string $name;    // string nullable
```

## Constantes

Las clases e interfaces pueden definir constantes mediante `const`, no mediante `define()`. Su visibilidad es `public` por defecto. Las clases tienen la constante `class` que contiene su nombre *fully qualified*.

Las constantes se almacenan en la clase, y no en las instancias. Por lo tanto su acceso se hace a través de la clase con el operador `::` del tipo `MiClase::class`. Desde dentro de la clase se debe acceder mediante `self::`.

## Autoload de clases

Una forma frecuente de trabajar es crear un archivo por cada clase. Lo malo es tener que indicar una larga retahíla de *includes* para cargar todas las clases al prinicipio de cada *script*. Para evitarlo, se puede registrar un (o más) *autoloader*. Se hace con `spl_autoload_register()`, al que se le pasa una función anónima que indica qué hacer cuando intentamos usar una clase que no está definida. Tal función recibirá un argumento con el nombre de la clase.

Por ejemplo:

```php
spl_autoload_register(function($nombre)
{
    require $nombre . '.php';
});
```

O lo que es equivalente (recordemos que un *callable* puede ser una *closure*, pero también un nombre de función):

```php
function autocarga($nombre) {
    require $nombre . '.php';
}
spl_autoload_register('autocarga');
```

A partir de aquí, si hacemos `$ob = new MiClase;` y ***MiClase*** no está definida, *PHP* echará mano del *autoloader* registrado, pasándole automáticamente como parámetro un *string* con el nombre de la clase (en este caso ***'MiClase'***), con lo que tal función ejecutará `require 'MiClase.php';` antes de ejecutar el `new`.

> Si la clase está dentro de un *namespace* no global (***MiEspacio\MiClase***), la ruta *full-qualified* será incluida en el nombre que se pasará a la función de autocarga, y será interpretada como la ruta del archivo a incluir.

## Constructor y destructor

El constructor es una función `__construct()` que no debe retornar nada. Puede tener tantos argumentos como se quiera. Los argumentos se pasarán en el momento de crear la clase con `new`.

Las clases hijas que definen constructor no llaman implícitamente al constructor del padre, con lo que si se quiere llamar desde este, hay que hacerlo mediante `parent::__construct()` pasándole los argumentos pertinentes. Si la clase hija no define constructor, lo hereda del padre (a no ser que el constructor del padre sea `private`).

El constructor *overriden* está exento de las reglas de compatibilidad.

A partir de *PHP* 8, los argumentos pasados al constructor pueden convertirse directamente en propiedades del objeto (*constructor property promotion*). Esto se hace incluyendo modificador de visibilidad en los parámetros que deseemos convertir en propiedades automáticamente.

```php
public function __construct(public int $x, protected $y, string $z) {}
```

En este caso, ***\$x*** e ***\$y*** se convertirán en propiedades del objeto. Tras ello, se ejecutará el código del cuerpo del constructor (que puede estar vacío).

El constructor podría ser privado, en cuyo caso solo podría ser llamado por un método estático de la clase, que debería crear y retornar el objeto:

```php
class MiClase
{
    private function __construct($a, $b) { /* ... */ }
    public static function construyeA(): static { $o = new static(0, 1); return $o; }
    public static function construyeB(): static { $o = new static(1, 1); return $o; }
    ...
}
$ob1 = MiClase::construyeA;
$ob2 = MiClase::construyeB;
```

Para los usos nuevos de `static`, ver el apartado de [*binding*](#binding-enlazado). En nuestro ejemplo, se refiere a la clase ***MiClase***.

En cuanto al **destructor**, se ejecuta cuando ya no hay referencias al objeto, o al finalizar la ejecución del *script*. No tiene parámetros, no retorna nada y se llama `__destruct()`.

## Visibilidad y herencia

La herencia es simple en *PHP*.

Las propiedades, métodos y constantes pueden tener visibilidad `public`, `protected` o `private`. Objetos de la misma clase tendrán acceso entre sí a sus elementos privados y protegidos, aunque no sean la misma instancia.

Es obligatorio definir la visibilidad de las propiedades. Un método sin modificador de visibilidad es público por defecto. Lo mismo sucede con las constantes.

Si se utiliza inicializador al declarar una propiedad y además el constructor le da valor, el constructor tiene preferencia.

Solo los elementos públicos son accesibles desde fuera de la clase.

Los elementos privados no se heredan. Se puede cambiar la visibilidad de un elemento heredado a una más débil, es decir, lo heredado protegido se puede redefinir como público.

Una clase debe estar definida antes de usarse (con excepción de los *autoloaders*).

## Operador de resolución de *scope* (::)

El operador `::` permite acceder a propiedades y métodos estáticos, a constantes y a métodos *overriden* de una clase.

Al usarse fuera de la clase, se debe usar el nombre de la clase, o una variable que contenga ese nombre. Desde el interior de la clase se usa con `self` (clase actual), `parent` (clase padre) o `static` (véase apartado sobre *binding*).

## static

A parte de lo dicho en [*binding*](#binding-enlazado), y para variables estáticas de una función, `static` sirve para declarar una propiedad o método como estático, es decir, perteneciente a la clase y no a la instancia.

El modificador `static` se puede añadir a cualquier método o propiedad de una clase.

## Abstracción de clases

Una clase definida como abstracta no se puede instanciar. Una clase que contenga por lo menos un método definido como abstracto debe ser abstracta. Los métodos definidos como abstractos no pueden definir la implementación del método, solo la *signature*.

Una clase derivada de una clase abstracta debe implementar todos los métodos abstractos que recibe, y tienen que ser compatibles con la *signature* del método del padre. Si no los implementa todos, y/o si declara nuevos métodos abstractos, debe declararse a su vez como clase abstracta.

```php
abstract class MiClase
{
    abstract function foo();
    abstract function bar();
}
```

## ¿Literales objeto?

En *PHP* no existen los literales objeto, como por ejemplo en *Javascript*. Sin embargo se puede conseguir un mecanismo similar utilizando literales *array* y haciendo el *cast* a tipo ***object***. En el siguiente ejemplo, obtenemos un *array* de 3 objetos:

```php
$arr = [
    (object)[ 'campo1' => 1, 'campo2' => 2 ],
    (object)[ 'campo1' => 11, 'campo2' => 22 ],
    (object)[ 'campo1' => 111, 'campo2' => 222 ]
];
```

Para acceder a los valores, podemos hacer:

```php
$v1 = $arr[1];  // retorna el segundo objeto
$v2 = $arr[0]->campo2;  // retorna el entero '2'
```

## Interfaces

Una interfaz se define igual que una clase, pero usando `interface` en lugar de `class`. No puede definir los métodos que tenga, si los tiene, y estos deben ser públicos. Una clase puede definirse para implementar una interfaz mediante `implements`.

Las interfaces pueden contener constantes y/o métodos, pero no propiedades. Las clases que implementen las interfaces no pueden cambiar el valor de las constantes, y deben implementar los métodos de esta, teniendo en cuenta la compatibilidad.

Las interfaces pueden heredar de otras interfaces, incluso con herencia múltiple. Véase el ejemplo:

```php
interface a { function foo(); function bar(); }
interface b { /* ... */ }
interface c extends a, b { /* ... */ }
class MiClase1 implements c { /* ... */ }
class MiClase2 extends MiClase1 implements a, b { /* ... */ }
```

Una clase que implemente una o más interfaces pero no defina todos los métodos, debe declararse como abstracta.

Si una clase implementa interfaces y extiende otra clase, debe indicarse `extends` primero.

## Traits

Usado para reutilizar conjuntos de métodos en un lenguaje en el que no disponemos de herencia múltiple. Los *traits* no pueden instanciarse, y no pueden contener constantes.

Un *trait* no puede actuar como clase base. Para usar un *trait* se debe indicar con `use` dentro de una clase o de otro *trait* (no dentro de una interfaz).

Un método definido en un *trait* tendrá prioridad sobre un método de igual nombre definido en una clase base, pero un método definido en la clase actual tiene prioridad sobre el mismo definido en un *trait*.

Se pueden usar varios *traits* en la misma clase, mediante una lista: `use t1, t2, t3;`.

A `use` se le puede añadir un bloque para configurar ese uso. Imaginemos que tanto el *trait* ***t1*** como el ***t2*** tienen un método ***foo()***. Usar ambos *traits* del modo `use t1, t2;` producirá error. Habrá que especificar cómo resolveremos esa duplicidad. Supongamos que queremos usar la versión de ***t2*** en nuestra clase. Se hace así:

```php
use t1, t2
{
    t2::foo insteadof t1;
}
```

Imaginemos que queremos usar ambas versiones. Una de las dos (o las dos) deberá tener un alias (por ejemplo, ***foot1()***) para poder referirnos a ella. O podemos cambiar la visibilidad de un método. O ambas cosas a la vez. Supongamos que uno de los dos *traits* tiene, además un método ***bar()*** y ***poo()***:

```php
use t1, t2
{
    t2::foo insteadof t1;    // resolución de conflicto
    t1::foo as foot1;    // resolución de conflicto mediante alias
    bar as protected;    // cambio de visibilidad
    poo as private poot1;    // cambio de visibilidad y alias
}
```

Los *traits* tienen acceso de la forma habitual a `$this`, `parent`, etc., ya que pueden definir métodos estáticos y no estáticos.

Pueden también declarar métodos abstractos. En este caso, en la clase que los implemente pueden tener distinta *signature* (ser incompatibles).

## Clases anónimas

Útiles, por ejemplo, para pasar como argumento un objeto que no necesita ser almacenado:

```php
foo(new class { /* ... */ });
foo(new class(5, 'hola') extends MiBase implements a, b { /* ... */ });
```

Si la clase anónima se crea dentro del código de otra clase, no tiene acceso a los miembros privados y protegidos de la clase exterior.

## Sobrecarga

Se le llama sobrecarga, **pero no tiene nada que ver**. Se trata simplemente de una serie de funciones que son llamadas al acceder a propiedades o métodos no accesibles (por su visibilidad o porque simplemente no existen). Su uso es *self-explanatory*:

```php
public __set(string $nombre, mixed $valor): void    // al dar valor a una propiedad no accesible
public __get(string $nombre): mixed    // al intentar obtener el valor de una propiedad no accesible
public __isset(string $nombre): bool    // al llamar a isset() o empty() sobre una propiedad no accesible
public __unset(string $nombre): void
```

Solo funcionan en contexto de objeto (no contextos estáticos).

En cuanto a los métodos:

```php
public __call(string $nombre, array $args): mixed    // al llamar a un método no accesible en contexto de objeto
public static __callStatic(string $nombre, array $args): mixed    // lo mismo pero en contexto estático
```

El *array* pasado a estas funciones es un *array* enumerado (claves 0, 1, 2,...).

## Iteración de objetos

Es posible iterar sobre los objetos (mediante `foreach`, por ejemplo). Por defecto, todas las propiedades visibles se usarán para las iteraciones.

Para ir más allá, se puede implementar la interfaz `Iterator` en la clase, permitiendo controlar cómo será la iteración. Los métodos de esta interfaz son:

```php
public current(): mixed    // retorna el elemento actual
public key(): scalar    // retorna la clave (índice) del elemento actual
public next(): void    // mueve una posición, al siguiente elemento
public rewind(): void    // mueve el iterador al primer elemento
public valid(): bool    // comprueba si la posición actual es válida
```

## Métodos especiales (magic methods)

Cambian la acción por defecto en ciertas acciones realizadas sobre un objeto o clase. Empiezan por dos guiones bajos (***\_\_***).

Todos los *magic methods* deben ser públicos, a excepción de `__construct`, `__destruct` y `__clone`.

Si se define un *magic method* hay que respetar su *signature*.

A parte de los vistos hasta ahora, existen otros métodos mágicos como `__toString()`, que controla en qué forma se debe convertir el objeto a *string*, o `__invoke()`, que controla cómo se comportará el objeto cuando sea llamado como si fuese una función.

```php
public __toString(): string
__invoke(...$valores): mixed
```

Por otro lado, tenemos:

```php
__debugInfo(): array
```

Este método será llamado cuando hagamos `var_dump()` sobre el objeto. Si no está definido, simplemente se mostrarán todas las propiedades (privadas, protegidas y públicas) del objeto.

## final

Los métodos definidos con el modificador `final` no pueden ser *overriden* en clases hijas. Si la clase es la que está definida como final, ninguna otra clase la puede usar como clase base.

Este modificador no afecta al rendimiento.

## Clonado de objetos

Para clonar un objeto se utiliza la palabra clave `clone`.

```php
$a = new MiClase;
$b = $a;    // $b hace referencia a la misma instancia que $a
$c = clone $a;    // $c se refiere a una nueva instancia, aunque inicialmente con el mismo contenido que $a
```

Al clonarse un objeto, se crea una *shallow copy*, es decir, se copia la instancia, pero las propiedades que hagan referencia a otros objetos seguiran referenciando a esos objetos.

Si la clase del objeto clonado tiene un método `__clone()`, tras el clonado, el método `__clone()` **del nuevo objeto creado** será ejecutado.

```php
__clone(): void
```

## Comparación de objetos

Una comparación por igualdad (`==`) de dos objetos devolverá verdadero si y solo si tienen los mismos atributos, sus valores son iguales (comparados por igualdad entre ellos), y si los dos objetos pertenecen a la misma clase. Al compararse por identidad (`===`), el resultado será verdadero si y solo si se refieren a la misma instancia.

## Binding (enlazado)

En un escenario con herencia y métodos *overriden*, cuando desde un método invocamos otro de los métodos de la clase, y este último tiene varias posibilidades (por ser heredada), es necesario saber a qué método estamos haciendo referencia cuando escribimos `$this`, es decir a qué clase pertenece el método que se debe ejecutar:

```php
class Base
{
    public function quesoy() { return "Soy de clase Base."; }
    public function show() { echo $this -> quesoy(); }
}

class Deriv extends Base
{
    public function quesoy() { return "Soy de clase Deriv."; }
}
```

En este ejemplo, la llamada a `$this->quesoy()` puede referirse a la versión de ***Base*** (si la instancia es de ese tipo) o la de ***Deriv*** (si la instancia es de tipo ***Deriv***). Esto solo se sabrá en el momento de ejecución, ya que la versión a utilizar **depende del objeto** llamante. La resolución se hará pues en *runtime*, y se denomina *late binding* o *dynamic binding*.

En un escenario sin herencia, por ejemplo, una invocación, desde dentro de un método a otro de los otros métodos, no tiene pérdida, y puede resolverse en tiempo de compilación, pues en ese momento ya sabemos a qué clase pertenece `$this`. Es lo que se llama *early binding* o *static binding*.

Pero hay un caso especial en *PHP*.

### Late static binding

En el caso anterior, la resolución se hacía en función de `$this`. Pero cuando estamos en un método estático no disponemos de `$this`, sino únicamente de `self::` (y `parent::`). Veamos el ejemplo anterior en versión estática:

```php
class Base
{
    public static function quesoy() { return "Soy de clase Base."; }
    public static function show() { echo self::quesoy(); }
}

class Deriv extends Base
{
    public static function quesoy() { return "Soy de clase Deriv."; }
}
```

En este caso, al hacer `Base::show()` se ejecutará (correctamente) el método `Base::quesoy()`. Pero si hacemos `Deriv::show();` también se ejecutará (incorrectamente) ese método heredado de la clase ***Base***, ya que la llamada se hace desde `Base::show()`, donde `self::quesoy()` significa `Base::quesoy()`. Es la limitación que tiene acceder a través de `self`. Para solucionarlo, disponemos del identificador `static`, que supera esta limitación, ya que no representa a la clase actual, sino a la clase que se utilizó para hacer la invocación en tiempo de ejecución. En este caso, el código correcto quedaría:

```php
class Base
{
    public static function quesoy() { return "Soy de clase Base."; }
    public static function show() { echo static::quesoy(); }
}

class Deriv extends Base
{
    public static function quesoy() { return "Soy de clase Deriv."; }
}
```

A través de `static::` solo podemos hacer referencia a propiedades y métodos estáticos.

## Serialización de objetos

La función `serialize()` sobre un objeto retorna un *string* con una representación del objeto en un *stream* de *bytes*. Mediante la función `unserialize()` se puede reconstruir el objeto serializado, pasándole ese *string*. Para deserializar un objeto necesitamos tener definida la clase de ese objeto en el momento de invocar `unserialize()`, usándolo en un contexto en el que quede claro a qué clase pertenece la deserialización.

## Reglas de varianza

Cuando hablemos de tipos más o menos específicos, consideraremos que un tipo es más específico en estos casos:

- Cuando se elimina uno de los tipos en una unión de tipos.
- Cuando una clase se cambia por una clase hija de esta.
- Cuando `float` se cambia a `int`.

En caso contrario, será un tipo menos específico.

### Covarianza

Permite al método *overriden* en el hijo, retornar un tipo más específico que el del padre. Esto incluye que el método del hijo puede definir un tipo de retorno si el padre no lo hace.

### Contravarianza

Permite que un tipo de parámetro sea menos específico en el método del hijo. Esto incluye que el método hijo puede no especificar tipo para parámetros cuyo tipo sí especifica el padre.
