# Estructuras de control

Una sentencia *PHP* termina en punto y coma (***;***). Las sentencias se pueden agrupar mediante llaves (***\{}***), formando una sentencia en sí.

## if

```php
if(<expresión>)
    <sentencia>  // puede ser una sentencia o grupo de ellas
```

La sentencia se ejecuta solo si la expresión es verdadera.

```php
if(<expresión>)
    <sentencia>
else
    <sentencia>
```

Si no se ejecuta la sentencia del bloque `if`, se ejecutará la del bloque `else` y al revés.

Puede haber uno o más bloques `elseif` (aunque como máximo un bloque `else`):

```php
if(<expresión>)
    <sentencia>
elseif(<expresión>)
    <sentencia>
elseif(<expresión>)
    <sentencia>
else
    <sentencia>
```

Solo se ejecuta un bloque `elseif` si no se ha ejecutado ninguno de los bloques anteriores y si su expresión es verdadera, es decir se ejecutará el primer `elseif` con expresión verdadera (siempre que no se haya ejecutado el bloque `if`). Solo puede ejecutarse uno de los bloques de esta estructura, tanto `if` como `elseif` como `else`. Este último solo se ejecutará si todas las expresiones de control han resultado falsas.

Las sentencias `if/elseif/else` pueden anidarse tanto como se desee.

## Sintaxis alternativa

Las estructuras `if`, `while`, `for`, `foreach` y `switch` disponen de una sintaxis alternativa. En esta sintaxis, y suponiendo sentencias compuestas entre llaves, la llave inicial se sustituye por dos puntos (***:***) y la llave final se sustituye por `endif`, `endwhile`, `endfor`, `endforeach` o `endswitch` (seguido de punto y coma) respectivamente.

```php
if(<expresión>):
    <sentencia>
else:
    <sentencia>
endif;
```

Esto resulta útil para incluir código *HTML* directamente. Aunque también se puede hacer usando llaves, así resulta más claro:

```php
<?php ?>
    <h2>Código HTML</h2>
    <p>Esto es código HTML</p>
<?php else: ?>
    <h2>Otro código HTML</h2>
    <p>Esto es otro código HTML</p>
<?php endif; ?>
```

No es posible combinar ambas sintaxis en una misma estructura.

## while

```php
while(<expresión>)
    <sentencia>
```

La sentencia se ejecutará una y otra vez hasta que la expresión se evalúe como falsa. La expresión se evalúa antes de iniciar la sentencia (antes de la siguiente iteración).

En sintaxis alternativa:

```php
while(<expresión>):
    <sentencia>
endwhile;
```

## do-while

Similar a `while`, pero en este caso, la evaluación de la expresión se realiza después de cada iteración, con lo que nos aseguramos de que se produce, por lo menos, una iteración.

```php
do
    <sentencia>
while(<expresión>);
```

En este caso no hay sintaxis alternativa.

## for

Tiene esta sintaxis:

```php
for(<expr1>; <expr2>; <expr3>)
    <sentencia>
```

La primera expresión se evalúa (ejecuta) antes del bucle. La segunda se evalúa antes de cada iteración: si es verdadera, se ejecuta el código del bucle, de lo contrario, el `for` termina. Después de cada iteración, se ejecuta la tercera expresión.

Cada una de las tres expresiones puede estar en blanco, o consistir en varias expresiones separadas por coma.

Si la expresión 2 está formada por varias expresiones, se ejecutarán todas, pero el valor resultante será el de la última de ellas. Si está vacía se considerará verdadera, con lo que el bucle correrá indefinidamente.

Admite sintaxis alternativa:

```php
for(<expr1>; <expr2>; <expr3>):
    <sentencia>
endfor;
```

## foreach

Permite iterar sobre *arrays* y objetos. Tiene estas dos formas:

```php
foreach(<expresión iterable> as $valor)
    <sentencia>

foreach(<expresión iterable> as $clave => $valor)
    <sentencia>
```

En el primer caso, se va pasando por todos los elementos del iterable, y en cada iteración, la variable especificada (en este caso ***\$valor***) va tomando sucesivamente los valores de los elementos. En el segundo caso, adicionalmente, la variable específica (en el ejemplo, ***\$clave***) irá tomando al mismo tiempo los valores de las distintas claves.

Si deseamos cambiar los elementos del *array* sobre el que iteramos, las asignaciones se deben hacer por referencia, del tipo `foreach($arr as &$valor)`. Si se hace así, después del bucle, la variable ***\$valor*** seguirá viva, como referencia al último elemento del *array*, con lo que se recomienda deshacer la referencia (con `unset()`).

La sintaxis alternativa tiene esta forma:

```php
foreach(<expresión iterable> as $valor):
    <sentencia>
endforeach;

foreach(<expresión iterable> as $clave => $valor):
    <sentencia>
endforeach;
```

## break y continue

`break` termina la ejecución de un bucle `for`, `foreach`, `while`, `do-while` o `switch`. Se le puede dar un parámetro entero, por defecto 1, indicando cuántas estructuras anidadas romperá.

`continue` termina la iteración actual de un bucle `for`, `foreach`, `while`, `do-while` o `switch`. También acepta opcionalmente un parámetro entero, por defecto 1, indicando qué nivel tiene que saltarse, en un escenario anidado. Pero puede ser útil en caso que, por ejemplo, el `switch` esté dentro de un bucle y queramos saltar a la siguiente iteración de ese bucle. En ese caso, mediante `continue 2` dentro del `switch` iteraremos en ese bucle exterior.

Una estructura `switch` se considera un bucle, pero técnicamente no lo es. Así, `continue` funcionará igual que `break` en este tipo de estructuras (aunque generará un aviso, pues es probable que sea un error de programación).

## switch

Es equivalente a una serie de sentencias `if`. Su sintaxis es:

```php
switch(<expresión>)
{
    case <exp1>:
        <sentencia(s)>
    case <exp2>:
        <sentencia(s)>
    ...
    default:
        <sentencia(s)>
}
```

La expresión inicial se ejecuta una sola vez. Luego se va comparando (mediante `==`) con las expresiones de las cláusulas `case`. Cuando se encuentra una coincidencia, se sigue la ejecución en ese punto, hasta el final del bloque `switch`. Si queremos interrumpir la ejecución antes de la siguiente cláusula `case` hay que insertar un `break` al final de la presente cláusula `case`. Si no hay coincidencia, no se ejecutará ningún código, a no ser que exista la cláusula opcional `default` (solo puede haber una), que se ejecutará en ese caso.

Cuando la expresión inicial es compleja, es mejor una estructura `switch` que una serie de `if`/`elif`, ya que en el primer caso, la expresión es evaluada una única vez.

Las cláusulas `case` pueden dejarse en blanco para agrupar series de valores que compartan el mismo código a ejecutar.

La sintaxis alternativa tendría este estilo:

```php
switch(<expresión>):
    case <exp1>:
        <sentencia(s)>
    case <exp2>:
        <sentencia(s)>
    ...
    default:
        <sentencia(s)>
endswitch;
```

En todo caso, es posible cambiar los dos puntos (***:***) de las cláusulas `case` por punto y coma (***;***).

## match (*PHP* 8)

Similar a `switch`. Tiene esta sintaxis:

```php
match(<expresión>)
{
    <expr1> => <ret_expr1>,
    <expr2> => <ret_expr2>,
    <expr3> => <ret_expr3>,
    ...
    default => <ret_default>
};    // obsérvese el ';' final
```

`match` va comparando cada expresión ***\<exprN>*** con la expresión inicial ***\<expresión>***. Diferencias con `switch`:

- `match` retorna un valor: cuando encuentra una coincidencia, simplemente retorna el valor de la expresión ***\<ret_exprN>*** correspondiente.
- En lugar de comparar mediante `==`, lo hace usando `===`.
- La ejecución no sigue a otras cláusulas: se ejecuta una sola expresión de retorno.
- Las expresiones deben ser exhaustivas. Al menos una debe coincidir.

Una expresión ***\<exprN>*** puede estar compuesta por varias expresiones separadas por comas. La expresión de retorno correspondiente se ejecutará si una de estas subexpresiones coincide. La cláusula `default` es opcional.

Si en lugar de `===` deseamos cualquier otro tipo de operador de comparación, la expresión inicial puede ser ***true*** y las expresiones de retorno pueden contener cualquier tipo de comparación.

## declare

Permite establecer directivas para un bloque de código. Se les puede dar valor a estas directivas mediante un literal (no constantes ni variables). El efecto de declare se puede referir a un bloque de código específico:

```php
declare(ticks=1)
{
    /* script entero */
}
```

O en *scope* global:

```php
/* aquí no afecta */
declare(ticks=1)
/* afecta a partir de aquí hasta el final del archivo */
```

`declare` afectará hasta el final del archivo si está en *scope* global. Si el archivo está incluido, no afectará al código del archivo que lo incluye.

Actualmente existen 3 directivas posibles: ***ticks***, ***encoding*** y ***strict_types*** (tipado estricto).

### ticks

Un *tick* es un evento que sucede cada N (a especificar con `declare(ticks=<N>)`) sentencias *tickables* dentro del bloque `declare`. No todas las sentencias son *tickables*. La función a ejecutar debe registrarse con la función `register_tick_function()`.

### encoding

Indica la codificación del *script*: `declare(encoding='ISO-8859-1');`. Al combinarse con *namespaces*, se debe declarar usando la sintaxis en *scope* global.

## return

Retorna el control al módulo llamador, opcionalmente pasando un valor: `return [$valor]`. Puede terminar una función o un *script* entero. En este último caso, el `return` se hará desde *scope* global: si el archivo estaba incluido, la ejecución sigue tras la inclusión, opcionalmente dando valor a la llamada a `include()` o `require()`. Si era el *script* principal, termina su ejecución.

No es necesario el paréntesis para el valor, pues `return` no es una función. Si se omite el parámetro se entiende ***null***.

## include y require

`include` incluye y evalúa el archivo especificado. Si se especifica con la ruta completa (relativa o absoluta), lo busca allí. Si no, lo busca en el *include path* especificado en la configuración (se puede ver con `get_include_path()`). Si no lo encuentra allí, buscará en el directorio donde está el *script*, y finalmente en el directorio de trabajo actual. Si no lo encuentra, se genera un aviso.

El archivo queda incluido en el *scope* donde está la línea llamante, como si se hubiera escrito en ese punto. Aunque las funciones y clases definidas en ese archivo incluido tienen *scope* global.

Al principio del archivo incluido, el *parser* pasa a modo *HTML*, por lo que es necesario volver a escribir en él la etiqueta `<?php` si corresponde. Al final del archivo, vuelve a modo *PHP*.

Si el archivo incluido no retorna un valor en sí mismo, el valor de la sentencia `include` será ***null*** en caso de fallo (junto con un aviso levantado), o 1 en caso de éxito.

Al no ser una función, no es necesario incluir paréntesis: `include <archivo>`. Así, esto es erróneo:

```php
if(include('vars.php') == true) {...}
```

Porque se interpreta como `include 'vars.php' == TRUE`, es decir, `include ('vars.php' == true)`, o sea `include('1')`. Lo correcto sería escribir, por ejemplo:

```
if((include 'vars.php') == true) { /* ... */ }
```

Si hay funciones o clases definidas en el archivo incluido, que están después de un posible `return`, se podrán usar igual desde el archivo principal.

Si se incluye más de una vez un archivo que define funciones y/o clases, se generará un error, debido a que se definirán más de una vez. Para evitarlo, se debe usar `include_once`, que actúa como `include` con la diferencia que si el archivo ha sido ya incluido, no lo vuelva a incluir, y simplemente retorna ***true***.

En cuanto a `require`, es igual que `include`, pero se produce un error si el archivo no se encuentra. En cuando a `require_once`, es como `include_once` pero generando error si no encuentra el archivo.

## goto

Para saltar a una etiqueta:

```php
goto <eti>;
...
<eti>:
...
```

No se puede saltar a un *scope* distinto (de/a una función, p.e.), ni al interior de un bucle o `switch` (sí se puede saltar fuera de ellos, por ejemplo para hacer un `break` multinivel).
