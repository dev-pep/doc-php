# Tipos

Tipos admitidos:

- 4 tipos escalares (`bool`, `int`, `float`, `string`).
- 4 tipos compuestos (`array`, `object`, `callable`, `iterable`).
- 2 tipos especiales (`resource`, `NULL`).

El tipo de la variable es decidido en tiempo de ejecución.

La función `var_dump()` indica el valor y tipo de una variable de forma legible. `gettype()` indica el tipo. Para comprobar el tipo en tiempo de ejecución, existen las funciones `is_int()`, `is_string()`, etc.

## boolean

Valor de verdad. Literales (*case-insensitive*) ***true*** y ***false***.

Para convertir explícitamente a `boolean` se usa el *cast* `(bool)` o `(boolean)`.

En cuanto a la conversión a booleano, se considera **falso**: el literal ***false***, los enteros 0 y -0, los flotantes 0.0 y -0.0, un *string* vacío o "0", un *array* con 0 elementos, el tipo `NULL` o las variables sin asignar.

Lo demás (incluido *NaN*) es ***true***.

## int

Puede representar enteros positivos, negativos y 0. Un literal se considera decimal, a excepción de:

- si empieza por ***0*** es octal.
- si empieza por ***0x*** (o ***0X***) es hexadecimal.
- si empieza por ***0b*** (o ***0B***) es binario.

Los enteros pueden contener guiones bajos (***\_***, no consecutivos) intercalados.

Un entero no genera *overflow*: se convierte en flotante.

Para convertir explícitamente a entero se usa el *cast* `(int)` o `(integer)`, o con la función `intval()`.

Desde un booleano, será 0 o 1. Desde un flotante, se redondea hacia 0; si excede la capacidad del entero, el resultado es indefinido (*NaN* o infinito pasan a 0). Desde *string*, si este contenía un valor numérico, pasa a tener ese valor; de lo contrario será 0. Desde ***null*** es siempre 0. Desde otros tipos, es indefinido.

## float

Números reales. Un literal flotante puede contener punto flotante, en cuyo caso se puede omitir la parte entera o la fraccionaria (pero no las dos). Además, puede contener exponente: ***e*** o ***E*** seguido de signo opcional y un número entero decimal.

> La precisión puede dar resultados inesperados. Por ejemplo, `(int)((0.1+0.7)*10)` dará 7 en lugar de 8, ya que es 7.999999... redondeado hacia abajo.

No se deben comparar flotantes por igualdad.

Un *string* se convierte al valor adecuado que represente, o a 0 si no. Cualquier otra conversión se hace convirtiendo primero a entero y de ahí a flotante.

Constantes ***NAN*** (*not a number*) e ***INF*** (infinito). ***NAN*** es distinto a todo, incluso a sí mismo, excepto a ***true***. Para saber si un valor es *NaN*, se debe usar `is_nan()`. En cuanto a infinito, por ejemplo ***-1/0*** es ***-INF***.

## string

Secuencias de caracteres (solo *bytes*, no *wide chars*).

### Single quoted

Literal *string* entre comillas simples. Los *newlines* del *string* forman parte de su contenido. Para indicar un carácter comilla simple dentro del *string*, hay que prefijar *backslash* (***\\'***). Para indicar barra invertida, también (***\\\\***). En cualquier otro caso, la barra invertida no tiene ningún significado especial.

### Double quoted

Entre comillas dobles se pueden indicar estos caracteres especiales: ***\\n*** (LF), ***\\r*** (CR) ***\\t*** (HTAB), ***\\v*** (VTAB), ***\\e*** (ESC), ***\\f*** (FF), ***\\"***, ***\\\\***, ***\\\$***.

También se puede indicar un carácter mediante una secuencia de 1 a 3 caracteres octales precedidos por *backslash* (***\\58***), ignorando el bit más significativo. También un número hexadecimal de 1 o 2 dígitos precedido por ***\\x*** (***\\x58***). También se puede indicar un *codepoint Unicode* mediante una secuencia de dígitos hexadecimales del tipo ***\\u\{1400}***: resultará en uno o más caracteres con la representación *UTF-8* de ese carácter.

Además, los nombres de variables son expandidos.

### Heredoc

Este tipo de *string* empieza con ***\<\<\<***, un identificador y *newline*. Luego el *string* en sí, y una línea de cierre que solo contiene el identificador (debe empezar en la columna 1) y el correspondiente ***;*** final. Este *string* se comporta igual que un *double quoted string*, con la diferencia de que no es necesario usar la barra invertida para indicar comillas dobles.

### Nowdoc

Una *nowdoc* es a las *single quoted strings* lo que una *heredoc* es a una *double quoted*.

### Parsing de variables

Se aplica solamente a *double quoted strings*, o a las *single quoted*.

#### Sintaxis simple

Es simplemente indicar el nombre de la variable, incluyendo el ***\$*** inicial. El *parser* tomará el nombre a continuación (de forma *greedy*).

```php
$juice = 'apple';
echo "I like $juices";    // produce: I like (no conoce $juices)
echo "I like ${juice}s";    // produce: I like apples
```

Con la primera forma también se pueden indicar elementos de un *array* o propiedades de un objeto.

La segunda forma se utiliza cuando queremos que el nombre de la variable a expandir no se tome de forma *greedy*. En ese caso, no puede haber espacios entre las llaves y el nombre de la variable, ni entre el signo ***\$*** y la llave de apertura.

#### Sintaxis compleja

Se puede incluir cualquier variable escalar, propiedad o elemento de *array*, entre llaves ***\{...}***.

Tanto en la sintaxis simple como compleja, el valor de estas referencias debe tener una representación *string*.

Para conseguir ***\{\\\$***, hay que escribir ***\{\\\\\$***.

Para elementos de un *array* con clave no numérica, solo se puede usar la sintaxis compleja.

```php
$fruta = "pera";
$nombre = "fruta";
echo "contenido de $nombre: {${$nombre}};"    // produce: contenido de fruta: pera
```

### Acceso (I/O) a los caracteres

Los caracteres de un *string* son accesibles mediante un *0-based index*. Índices negativos indican *offset* desde el final. Se suelen usar corchetes (***[N]***) para ello.

Escribir caracteres fuera de los límites del *string*, rellena con espacios. El índice debe ser entero o tener forma de entero (***"1"*** es correcto, pero ***"1.0"*** no).

### Operadores y funciones

El operador para concatenar *strings* es el punto (***.***). Se puede concatenar la constante ***PHP_EOL*** que indica un fin de línea.

Funciones útiles: `substr()` extrae un fragmento, `substr_replace()` reemplaza un fragmento, `strlen()` indica la longitud del *string*.

### Conversión a string

Para convertir a *string* se puede usar el *cast* `(string)` o la función `strval()`.

Las expresiones se convierten a *string* implícitamente cuando son necesarias, tal como en la función `echo`, o cuando una variable se compara con un *string*.

El valor ***true*** se convierte a ***"1"***, mientras que ***false*** se convierte al *string* vacío. Los valores numéricos se convierten de manera acorde, incluyendo exponente, si procede. Un *array* se convierte al *string* ***"Array"***. Para convertir un objeto a *string* hay que usar el método del objeto `__toString()`. ***null*** se convierte siempre al *string* vacío.

Los objetos se pueden convertir a *strings* mediante serialización con la función `serialize()`.

### Detalles de los strings

Los *strings* se almacenan mediante una serie de *bytes* y un número indicando la longitud (no son *null-terminated*).

Si hay literales *string* en un *script .php* con caracteres *Unicode*, serán codificados usando la misma codificación del *script*.

### Strings numéricos

*String* que puede interpretarse como un entro o flotante. Cuando sea necesario ello, si es posible interpretarlo como entero, se resolverá como entero.

Si el *string* empieza con un número pero tiene otro contenido posterior, puede convertirse a ese valor numérico, pero se genera un aviso.

## Arrays

Mapa de pares clave-valor. Un elemento del *array* puede ser otro *array* (*array* multidimensional). Para hacer un volcado de un *array* podemos usar `var_dump()`, pero la función `print_r()` formateará el valor adecuadamente.

### Creación

```php
$arr = array(key1 => val1, key2 => val2,...)

$arr = [key1 => val1, key2 => val2,...]
```

Las claves pueden ser *string* o entero (positivo, negativo o 0). Los valores, cualquier tipo. Si es *string* y contiene un número decimal correcto (que no empiece por ***+***), será convertido a entero. Los flotantes se convierten a entero (truncando). Los booleanos, pasan a entero 0 o 1. ***NULL*** se convierte al *string* vacío. Si varios elementos usan la misma clave, solo será accesible el último de ellos.

Cuando las claves no son todas enteras, se le llama *array* asociativo, en contraposición a un *array* numérico o secuencial.

Si se omite la clave en algún elemento, se tomará el valor de la mayor clave entera hasta el momento, más uno (como mínimo será 0). Si se omite para el primer elemento, será 0. Ese valor de la mayor clave entera puede no existir ya. Ese valor se restablece al reindexar el *array*.

### Acceso

El acceso es mediante clave entre corchetes (***[]***).

Si una función retorna un *array*, se puede indexar también: `funcion()[$i]`.

Acceder a una clave inexistente, produce el mismo efecto que acceder a una variable indefinida (retorna ***null***).

### Creación/modificación

```php
$arr[5] = "pepe";    // si no existe el array, lo crea
$arr[-5] = "giuseppe";    // añade o modifica clave -5
$arr[] = "joe";    // en este caso, equivale a $arr[6]="joe"
```

Para eliminar un elemento: `unset($arr[$clave])`. Para eliminar el *array* entero: `unset($arr)`.

Si tras eliminar algún elemento se quiere reindexar el *array*, se puede usar `array_values()`, que renumera los enteros.

> El mecanismo de tratamiento de constantes no definidas hace que algo como `$arr[foo]`, anteriormente a *PHP* 8, funcione cuando no existe ninguna constante ***foo***. Lo que hace *PHP* es sustituir las constantes no definidas por *strings* con ese contenido, con lo que en este caso quedaría `$arr['foo']`. Sin embargo se genera un aviso. A partir de *PHP* 8, levanta un error.
>
> Sin embargo, dentro de *string*, este mecanismo funciona bien siempre, sin aviso, ya que dentro de *strings* no se miran las constantes: `"string con $arr[foo]"`. Hay que hacerlo así obligatoriamente. Si la variable va entre llaves, entonces sí hay que indicarlo de la forma habitual: `"string con {$arr['foo']}"`.

### Conversión

Si un tipo escalar se convierte a *array*, resulta un *array* con un solo elemento e índice 0. Es decir, `(array)$escalar` equivale a `array($escalar)`.

Si se convierte a *array* un objeto compuesto, resultará un *array* con las propiedades de este como elementos. Sus claves serán los nombres de las propiedades y los valores sus valores. Los nombres de las propiedades privadas llevan como prefijo el nombre de la clase entre caracteres nulos, y los de las *protected* llevan el prefijo ***\"\\0\*\\0"***.

La conversión de un objeto `NULL` produce un *array* vacío.

## Iterables

Se construyen en base a un *array*, o un objeto que implemente la interfaz `Traversable`. El objeto resultante se puede iterar con `foreach`.

## Objetos

Se crean mediante `new`.

Si se convierte un valor cualquiera a objeto (con un *cast* a `object`, por ejemplo), se crea un instancia de la clase `stdClass`. Si es desde `NULL`, esta nueva instancia estará vacía. Si se convierte desde *array* resultará un objeto con las propiedades indicadas por las claves, con sus valores correspondientes.

Un escalar se convierte a objeto con una propiedad llamada `scalar`.

## Recursos

El tipo `resource` almacena una referencia a un recurso externo. Archivos, conexiones a bases de datos, lienzos de imagen, etc. Se usan mediante funciones especializadas. Al liberarse el recurso, entra en juego el *garbage collector* (con la excepción de las conexiones persistentes).

## NULL

Solo tiene un posible valor: la constante ***null*** (*case insensitive*).

Una variable tiene ese valor si le es asignado directamente, si no se ha asignado valor todavía, o si ha sido desasignada con `unset()`.

## Invocables

El tipo `callable` permite pasar un objeto invocable (ejecutable) como parámetro a las funciones que lo requieren.

- Para pasar como parámetro una función cualquiera, se hace mediante un simple *string* con su nombre.
- Para pasar un método estático, se pasa un *array* con el nombre de la clase (índice 0) y el nombre del método (en el índice 1). Alternativamente, se puede pasar un solo *string* con la notación ***'Clase::Metodo'***. Si por ejemplo la clase ***D*** es una derivada de la clase base ***B***, y queremos pasar por parámetro un método estático ***smet()*** de la clase ***B***, desde la clase ***D***, el *array* puede ser `array('D', 'parent::smet')`
- Si queremos pasar el método de una instancia, usamos un *array* con el objeto instanciado (en el índice 0) y un *string* con el nombre del método (índice 1).
- En general, cualquier objeto que implemente `__invoke()` se puede pasar como *callback*. Se pasaría el objeto en sí.
- Una función anónima se pasaría directamente.

## Declaración de tipos

Pueden añadirse declaraciones en los parámetros (todos o algunos) y/o valor de retorno de una función, así como en las propiedades de una clase.

> Al *override* un método, se debe respetar el mismo tipo de retorno que en la clase padre. Si el padre no lo define, el hijo puede hacerlo.

Si especificamos una clase, se aceptan objetos de esa clase o de cualquier derivada.

Si especificamos una interfaz, se aceptan objetos que pertenezcan a una clase que implemente esa interfaz.

```php
function sum(int $a, int $b, $c): float
{ ... }
```

Este ejemplo retornará siempre un flotante.

Si prefijamos ***?*** al tipo declarado significa que se acepta ***null*** también. Es lo que se denomina un tipo *nullable*.

### Unión de tipos (*PHP* 8)

Se usan para permitir más de un tipo. Son de la forma `T1|T2|...`. Uno de los tipos puede ser `NULL`, pero este no puede ser el único tipo.

En el **valor de retorno** se pueden definir los tipos `void` (no retorna valor, debe ser el único tipo) o `static` (se refiere a clase a la que pertenece el método).

#### mixed

`mixed` equivale a la unión `array|bool|callable|int|float|object|resource|string|null`.

## Tipado estricto

Por defecto, *PHP* cambia automáticamente un tipo erróneo a uno escalar correcto, si es posible. Pero se puede declarar el tipado estricto en un archivo *.php* mediante `declare(strict_types=1)`. El tipado estricto se aplica a las **llamadas** hechas en el archivo, no a las funciones declaradas en él (al definir una función es posible declarar el tipo de los parámetros y/o del valor de retorno).

## Malabarismos de tipos (*type juggling*)

El tipo de una variable se determina por el contexto. Se puede forzar el tipo de una variable con `settype()`. Para evaluar una variable como un cierto tipo, se usa el *type casting*.

El *type casting* se realiza prefijando el tipo deseado entre paréntesis.
