# Constantes

Identificador cuyo valor no puede cambiar. *Case-sensitive*. Como las variables, pero sin el ***$***. Por convenio, todo mayúsculas.

Se definen así:

```
define(string $nombre, mixed $valor);
```

o así:

```
const nombre = $valor;
```

En el caso de `const` solo se le puede dar un valor escalar. En este caso, la definición debe hacerse en el *scope* global, ya que el valor se decide en tiempo de compilación, mientras que con `define` se resuelve la expresión en *runtime*.

Las constantes son accesibles desde cualquier parte (tienen *scope* global). Una vez definidas no pueden cambiarse ni aplicarse `undef()`.

Para una lista de las constantes definidas, se usa `get_defined_constants()`. Para comprobar si una constante está definida, `defined()`.

Para acceder a su valor, simplemente se debe escribir su nombre, o usar `constant()`. La última forma tiene la ventaja que no hay que especificar el nombre de la constante en el código, ya que se le pasa un *string* con dicho nombre (se decide en tiempo de ejecución).

## Constantes predefinidas

Existe una serie de constantes que están predefinidas. Por un lado, las *core predefined constants* están siempre disponibles. Dependiendo de las extensiones instaladas, habrá otras tantas disponibles.

Algunos ejemplos de constantes *core* pueden ser ***PHP_VERSION***, ***PHP_DEBUG***, ***PHP_EOL***, , ***PHP_INT_MAX***, ***PHP_INT_MIN***, ***true***, ***false***, ***null***, etc.

## Constantes mágicas

Las *magic constants* pueden ir cambiando a lo largo de la ejecución, con lo que no son realmente constantes, aunque no se les puede asignar valor. Son *case insensitive*:

- ***\_\_LINE__*** indica la línea del *script*.
- ***\_\_FILE__*** es la ruta completa del archivo fuente (si es un archivo incluido, es la ruta del archivo incluido, no del que lo incluye).
- ***\_\_DIR__*** es el directorio del archivo fuente. Equivale a `dirname(__FILE__)`.
- ***\_\_FUNCTION__*** devuelve el nombre de la función actual. Si es una función anónima retorna ***\"{closure}\"***.
- ***\_\_CLASS__*** es el nombre de la clase actual, incluyendo el *namespace* en el que está definida (tipo ***\"Namesp\\Clase\"***). Si se usa en un *trait* devuelve el nombre de la clase que usa ese *trait*.
- ***\_\_TRAIT__*** es el nombre del *trait* actual, incluyendo el *namespace* en el que está incluido (tipo ***\"Namesp\\Trait\"***).
- ***\_\_METHOD__*** es el nombre del método actual.
- ***\_\_NAMESPACE__*** es el nombre del *namespace* actual.
- ***ClassName::class*** es el nombre de la clase actual *fully qualified*.
