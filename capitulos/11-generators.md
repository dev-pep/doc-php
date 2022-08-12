# Generators

Un generador permite crear un objeto iterador sin tener que crear una clase que implemente la interfaz ***Iterator***.

Un iterador es un objeto que utiliza `foreach` para iterar.

Un generador no precisa la creación de un *array* de elementos. Es una simple función que va retornando valores generados mediante `yield`. Puede retornar al finalizar el código, o mediante `return`.

Al llamar a una función generador se crea (y retorna) un objeto ***Generator***, que lógicamente implementa la interfaz ***Iterator***.

```php
function generaNums($inicial, $final)
{
    for($i = $inicial; $i <= $final; $i++)
        yield $i;
    return 'Todo bien';
}

$generador = generaNums(5, 50);  // creamos el objeto
                                 // generador
foreach($generador as $num)
{
    /* ... */
}
```

Una vez **terminadas todas las iteraciones**, si el generador retorna un valor (mediante `return`), puede obtenerse el valor retornado mediante el método `getReturn()` del generador.

A cada valor retornado se le asocia una clave entera *zero-based* autoincrementada, como en un *array* no asociativo. Si deseamos retornar clave y valor, se puede hacer un `yield $key => $value;`.

Una sentencia `yield` sin argumento retorna ***null***.

También puede retornar por referencia, como `return`.

El generador puede ir retornando valores, y en un momento dado, empezar a retornar (`yield from`) valores desde otro objeto. Este otro objeto debe ser de un tipo que implemente la interfaz ***Traversable***, es decir, iterable con `foreach`. Esta interfaz la implementa la clase ***Iterator*** y la ***IteratorAggregate***. Un posible problema es que las claves asociadas a los valores devueltos serán las que vengan del *traversable* en cuestión. En este sentido, si queremos convertir el generador en un *array*, podemos usar la función `iterator_to_array()`, la cual tiene un segundo parámetro, que si se indica ***false*** ignora las claves retornadas y las reasigna a una clave entera *zero-based* uniformemente autoincrementada.

Los generadores son más simples de construir que una clase para un iterador. Pero los generadores solo pueden iterarse de principio a fin, sin retroceder, y una sola vez.
