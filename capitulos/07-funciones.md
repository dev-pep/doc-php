# Funciones

La sintaxis para definir una función es:

```
function foo($arg1, $arg2,...)
{
    /* código */
    return $valor;
}
```

Está permitido anidar funciones e incluso clases. Las funciones pueden ser definidas después de ser referenciadas, a no ser que esté definida en un bloque condicional (dentro de un `if`, por ejemplo). Todas las funciones y clases en *PHP* tienen *scope* global. Sin embargo, para que una función sea visible, debe estar definida en *scope* global, ya sea antes o después de su referencia; en caso contrario, debe haberse ejecutado ya su definición. Por ejemplo:

```
bar();    // ERROR! No está definida en scope global ni se ha ejecutado su definición
poobarboo();    // OK! Está definida más abajo en scope global
bar();    // OK! Al llamar a poobarboo() arriba, se ha ejecutado su definición

function poobarboo()
{
    function bar() { echo "BAR"; }
    poo();    // OK! Está definida más abajo en scope global
    bar();    // OK! Ya se ha ejecutado su definición más arriba
    boo();    // ERROR! No está definida en scope global ni se ha ejecutado su definición
    function boo() { echo "BOO"; }
    boo();    // OK! Ya se ha ejecutado su definición arriba
}

function poo() { echo "POO"; }
```

No se permite sobrecarga, ni redefinición de funciones.

Los nombres de funciones son *case-insensitive*.

## Argumentos

Los argumentos se evalúa de izquierda a derecha. Si un parámetro tiene valor ***null*** por defecto, lógicamente admitirá el valor ***null*** (*nullable* argument). Para que el argumento sea *nullable* se recomienda definirlo como tal, sin tener que darle valor nulo por defecto.

### Paso por referencia

Por defecto, los argumetos se pasan por valor. Para que el paso sea siempre por referencia, se debe prefijar ***&*** en el nombre de variable en la definición de la función. En este caso, pasarle un valor leventará error.

### Valores por defecto

Los parámetros (incluso los pasados por referencia) pueden tener valores escalares, *array* o ***null*** por defecto. El valor debe ser una expresión constante.

```
function foo($a = 'Valor')
```

Los parámetros por defecto deberían ir después de los parámetros obligatorios, de lo contrario, el funcionamiento puede ser incorrecto.

### Número variable de parámetros

Para definir funciones variádicas se usa un parámetro al que se le prefija el *token* `...`, de tal modo que recogerá todos los argumentos de la llamada. No puede haber más parámetros después de este.

```
function foo($parm1, $parm2, ...$parms)
{
    /* $parms es un array */
}
```

Se puede hacer una declaración de tipo a este parámetro, de tal modo que ese tipo se aplicará a todos los argumentos que recoja. También se puede definir que sean pasados por referencia prefijando ***&***:

```
function foo($parm1, $parm2, int &...$parms)
```

El *token* `...` también puede usarse en una llamada a función para desempaquetar el contenido de un *array* en argumentos de llamada:

```
$arr = [1, 2, 3];
echo add(...$a);
```

equivale a `echo add(1, 2, 3);`.

### Argumentos con nombre (*PHP* 8)

**En la llamada** se puede especificar cada argumento por su nombre. Esto será tenido en cuenta, en lugar del orden en que son incluidos:

```
foo(parm2: 33, parm1: 'hello');
```

Se pueden combinar los argumentos con nombre con argumentos posicionales, pero en ese caso los posicionales deben ir antes de los argumentos con nombre.

## Retorno de valores

Si la función omite `return`, el valor retornado será ***null***.

Si la función debe retornar una referencia, se debe prefijar con ***&*** (tanto en la definición como en la llamada):

```
function &foo()
{
    /* ... */
    return $unareferencia;
}

$a = &foo();
```

## Funciones variables

Las funciones se pueden llamar a través de una varible que contenga su nombre:

```
function foo() { /* ... */ }
$a = 'foo';
$a();
```

Si ***$ob*** es un objeto de una clase que tiene un método ***met()***, se puede invocar:

```
$a = 'met';
$ob->a$();
```

## Funciones anónimas

También conocidas como *closures*. Pueden pasarse como argumento, o asignarse como valor a una variable.

```
$a = function($n1, $n2)
{
    return $n1 - $n2;
};

$a(4, 2);
```

En el momento de definirse, la función tenía acceso a una serie de variables. Si especificamos convenientemente que use esas variables, podemos acceder a ellas:

```
$ms = "hola";
$fun = function() { echo "Resultado: " . $ms . "<br/>"; };
$fun();    // '': no hemos especificado que usa $ms
$fun = function() use ($ms) { echo "Resultado: " . $ms . "<br/>"; };
$fun();    // 'hola'
$ms = "adiós";
$fun();    // 'hola': contenido del momento en que se definió
$ms = "hola";
$fun = function() use (&$ms) { echo "Resultado: " . $ms . "<br/>"; };
$fun();    // 'hola'
$ms = "adiós";
$fun();    // 'adiós': la función usa ahora una referencia a la variable
```

Al especificar `use` debemos indicar entre paréntesis la lista de varibales separadas por comas.

## Funciones *arrow*

Son también *closures*. Se definen así:

```
$a = foo($n1, $n2) => $n1 - $n2;
```

En este caso, no es necesario `use`, ya que lo hace por defecto (por valor) de todas las variables accesibles.
