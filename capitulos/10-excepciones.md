# Excepciones

Las excepciones se lanzan automáticamente en *PHP* cuando se produce un error. Una excepción es una instancia de una clase que extiende la clase ***Exception***.

Insertando código dentro de un bloque `try` permite capturar excepciones levantadas en dicho código, mediante bloques `catch`. Cada bloque `try` debe tener por lo menos un bloque `catch` (puede haber más de uno) o `finally` (máximo uno).

Si una excepción se levanta en un *function scope* que no la recoge (con `catch`), la excepción irá ascendiendo (*bubble up*) por el *calling stack* hasta que sea recogida por un bloque `catch` coincidente. En su camino irá ejecutando todos los bloques `finally`. Si llega al *scope* global sin ser recogida, el programa terminará inmediatamente, con error fatal (a no ser que exista un manejador global de excepciones).

Para establecer un manejador global de excepciones, se usa `set_exception_handler()`, al que se le pasará el manejador, que debe ser un objeto llamable (*callable*). Si el argumento es ***null***, el manejador pasará a su estado por defecto. En todo caso, la ejecución del programa termina tras la ejecución de tal manejador. Este, a su vez debe aceptar un argumento: una instancia de una clase que implemente la interfaz ***Throwable***, es decir, ***Error***, ***Exception***, o una derivada de estas.

Es posible lanzar una excepción mediante `throw`:

```php
throw $exc;
```

Donde el argumento debe ser una instancia de ***Exception*** o derivada:

```php
throw new OutOfRangeException;
```

Cada bloque `catch` recibe (maneja) un tipo de excepción (o más de uno separándolos con el signo *pipe*). Desde *PHP 8* no es obligatorio especificar una variable (en tal caso no tendremos acceso al objeto excepción).

> En caso de varios bloques `catch` que capturen excepciones de tipos derivados, hay que indicar el tipo más específico primero, ya que una excepción de tipo derivado entrará en el primer bloque `catch` que contenga su misma clase o una superclase de ella.

Tras los bloques `catch` se puede especificar un bloque `finally`. El código en este se ejecutará siempre, independientemente de si se levanta excepción o no. Si se llega a un `return` dentro de un bloque `try` o `catch`, se evalúa el valor a retornar, tras lo cual se ejecuta el bloque `finally`, tras lo cual se ejecuta tal retorno, aunque si el bloque `finally` contiene a su vez un `return`, lo que se retornará será lo que se indique allí.

Es posible crear nuestras propias excepciones, derivando una clase de ***Exception***, o de una derivada. En todo caso, si modificamos su constructor, es importante que este llame al constructor de la clase padre.
