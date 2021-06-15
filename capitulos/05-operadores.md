# Operadores

## Notas sobre expresiones

El operador ternario es como en *C*: `$a ? $b : $c;`.

## Precedencia de operadores

Operadores con la misma precedencia juntos se evalúan según su asociatividad. Por ejemplo, `+` y `-` tienen la misma precedencia y asociatividad izquierda, con lo que `1 + 2 - 3` equivale a `(1 + 2) - 3`. En cambio, `=` tiene asociatividad derecha, con lo que `$a = $b = 5;` equivale a `$a = ($b = 5)`.

No pueden estar juntos dos operadores sin asociatividad (`1 < 2 > 1` no es legal).

Veamos una tabla de operadores, ordenados de mayor a menor precedencia (los operadores en la misma fila tienen la misma precedencia):

| Asociatividad  | Operadores     |
| :------------- | :------------- |
|                | `clone` `new`  |
| Derecha        | `**`           |
|                | `++` `--` `~` `(int)` `(float)` `(string)` `(array)` `(object)` `(bool)` `@` |
| Izquierda      | `instanceof`   |
|                | `!`            |
| Izquierda      | `*` `/` `%`    |
| Izquierda      | `+` `-` `.`    |
| Izquierda      | `<<` `>>`      |
|                | `<` `<=` `>` `>=` |
|                | `==` `!=` `===` `!==` `<>` `<=>` |
| Izquierda      | `&`            |
| Izquierda      | `^`            |
| Izquierda      | `\|`           |
| Izquierda      | `&&`           |
| Izquierda      | `\|\|`           |
| Derecha        | `??`           |
| Izquierda      | `? :`          |
| Derecha        | `=` `+=` `-=` `*=` `**=` `/=` `.=` `%=` `&=` `|=` `^=` `<<=` `>>=` `??=` |
|                | `yield from`   |
|                | `yield`        |
|                | `print`        |
| Izquierda      | `and`          |
| Izquierda      | `xor`          |
| Izquierda      | `or`           |

La precedencia indica cómo se agrupan los términos de una expresión, pero **no** el orden de evaluación.

## Operadores aritméticos

Los operadores `+` y `-` se pueden usar como unarios (para definir el signo) o binarios. Los operadores `*`, `/`, `%` (módulo) y `**` (elevar a una potencia) son siempre binarios.

El operador `/` retorna un *float*, a no ser que los operandos sean enteros (o *strings* representando enteros) y el primer operando sea divisible por el segundo, en cuyo caso retornará un entero. Para la división entera puede usarse `intdiv()`.

Los operandos del operador `%` se truncan a enteros. El resultado tiene el mismo signo que el primer operando. Para el módulo *float*, se puede usar `fmod()`.

## Operadores de asignación

El valor de una expresión de asignación es el valor asignado.

Los operadores binarios tienen su correspondiente operador de asignación: `+=`, `-=`, `*=`, etc. Se deben usar siempre que se pueda, porque mejoran la rapidez.

La asignación crea una copia de la expresión a asignar, con la salvedad que la asignación entre objetos se hace por referencia. Si se quiere asignar una copia se debe usar la palabra clave `clone`.

### Asignación por referencia

Si se quiere crear una referencia en lugar de una copia, de debe usar la sintaxis:

```php
$refvar = &$original
```

## Operadores *bitwise*

Estos operadores actúan sobre los *bits* del valor.

- `$a & $b` realiza la operación AND *bit* a *bit* en los dos operandos.
- `$a | $b` lo mismo con OR.
- `$a ^ $b` lo mismo con XOR.
- `~ $a` aplica NOT a cada *bit*.
- `$a << $b` desplaza a la izquierda los *bits* de ***$a*** tantas posiciones como indique ***$b***.
- `$a >> $b` desplaza a la derecha los *bits* de ***$a*** tantas posiciones como indique ***$b***.

En las operaciones AND, OR y XOR, si los operandos son *strings*, las operaciones se realizan sobre los caracteres *ASCII* y el resultado es otro *string*. En cualquier otro caso, los operandos se convierten a entero y el resultado es un entero. Algo equivalente sucede con el unario NOT.

En las operaciones de desplazamiento, los *bits* que sobrepasan se descartan (los enteros tendrán típicamente 32 o 64 *bits*). En el desplazamiento a la izquierda se insertan ceros a la derecha, y se pueden perder los bits de la izquierda (puede perderse el signo). En el desplazamiento a la derecha, se inserta el bit de signo por la izquierda. Los operandos y resultados son siempre tratados como enteros.

## Operadores de comparación

Los operadores de comparación son como en *C*, pero además existen `===` (igual valor e igual tipo) y `!==` (distinto valor o distinto tipo).

Los operadores `!=` y `<>` son equivalentes.

El operador *spaceship* `$a <=> $b` retorna un entero negativo cuando ***$a*** es menor que ***$b***, un entero positivo en caso contrario, y cero si son iguales.

Cuando los operandos son *strings* numéricos o uno es un número y el otro un *string* numérico, las comparaciones se hacen numéricamente. En el caso de los operadores `===` y `!==` no hay conversión previa de tipos.

Según los tipos, las reglas de comparación son, en orden de aplicación:

| Operando 1     | Operando 2     | Acción               |
| :------------- | :------------- | :------------------- |
| ***null*** o *string* | *string* | Se convierte el ***null*** al *string* vacío. |
| ***null*** o `bool` | cualquiera | Se convierten ambos lados a `bool` (***false*** < ***true***). |
| objeto         | objeto         | Las clases pueden definir sus comparaciones, pero siempre entre clases iguales; clases distintas son incomparables. |
| *string*, recurso, entero o flotante | *string*, recurso, entero o flotante | Se convierten *strings* y recursos a números. |
| *array*        | *array*        | El *array* con menos miembros es menor. Si son iguales, se comparan elemento a elemento (si una clave de un operando no se encuentra en el otro, son incomparables). |
| objeto         | cualquiera     | El objeto es siempre mayor. |
| *array*        | cualquiera     | El *array* es siempre mayor. |

Dos *floats* no deberían compararse por igualdad.

## Operador de control de errores

El operador `@` como prefijo de **una expresión**, suprimirá cualquier mensaje diagnóstico que pueda producirse en la evaluación de la expresión.

## Operador de ejecución

Mediante *backticks* (***\`\`***), *PHP* ejecuta el contenido de estas como comando *shell* y retornará el valor devuelto por el comando. Equivale a `shell_exec()`.

## Operadores de incremento y decremento

Los operadores incrementales `++` y `--` pueden ser *prefix* y *postfix*, igual que en *C*.

Al aplicarse en *strings*, no tienen ningún efecto. Como excepción, aplicar el operador de incremento (`++`) en un *string* que contenga únicamente letras y/o números, incrementará el contenido del *string* a forma de contador. Por ejemplo al incrementar ***\"A\"*** obtendremos ***\"B\"***, sobre ***\"f\"*** obtendremos ***\"g\"***, en ***\"Z\"*** obtendremos ***\"AA\"*** o en ***\"1z\"*** tendremos ***\"2a\"***.

Sobre booleanos no tiene efecto.

## Operadores lógicos

Podemos utilizar las operaciones lógicas ***AND*** (`&&` o `and`), ***OR*** (`||` o `or`), ***XOR*** (`xor`) o ***NOT*** (`!`) como en *C*. El motivo de tener duplicidad de operadores para ***AND*** y ***OR*** es porque actúan en distintos niveles de precedencia.

## Operadores para *strings*

Disponemos del operador de concatenación `.`, y su correspondiente operador `.=` de asignación.

## Operadores para *arrays*

Sean ***$a*** y ***$b*** dos *arrays*:

- `$a + $b` es la **unión** de ambos. Si hay claves duplicadas, se accederá al elemento de ***$a***.
- `$a == $b` será verdadero si ambos tienen los mismos pares clave/valor.
- `$a === $b` será verdadero si ambos tienen los mismos pares clave/valor, en el mismo orden y son del mismo tipo.
- `$a != $b` y `$a <> $b` serán verdaderos si `$a == $b` es falso.
- `$a !== $b` será verdaderos si `$a === $b` es falso.

## Operador de tipo

El operador `instanceof` comprueba si una variable es una instancia de una clase determinada: `$a instanceof MiClase` retorna verdadero si ***$a*** es instancia de ***MiClase*** o de una derivada de ***MiClase***. También se puede usar para comprobar si la variable es una instancia de una clase que implemente una determinada interfaz (` $a instanceof MiInterface`).

Para comprobar si la variable pertenece a la clase, el segundo operando puede ser el nombre de la clase, o un objeto de esa clase, o un *string* con el nombre de esa clase.
