# Constantes

Identificador cuyo valor no puede cambiar. *Case-sensitive*. Como las variables, pero sin el ***\$***. Por convenio, todo mayúsculas.

Se definen así:

```php
define(string $nombre, mixed $valor);
```

o así:

```php
const nombre = 'Pepe';
```

En el caso de `const` solo se le puede dar un valor escalar constante (número, *string* u otras constantes), mientras que con `define()` acepta cualquier tipo de expresión (constante o no).

Con `const`, el valor se decide en tiempo de compilación, mientras que con `define()` se resuelve la expresión en *runtime*.

Las constantes son accesibles desde cualquier parte, independientemente del *scope* en el que estén definidas. Una vez definidas no pueden cambiarse ni aplicarse `undef()`.

Como excepción a esto, **dentro de una clase** se puede definir una propiedad constante mediante `const` (no con `define()`). En este caso, la constante pasa a formar parte del *scope* dicha clase. Hay que tener en cuenta que una constante definida así es un miembro estático, con lo que no puede accederse a través de instancia, sino de clase. Adicionalmente, se le puede dar modificador de acceso (por defecto es público). Así, suponiendo una propiedad constante pública ***miConst*** de la clase ***MiClase***, desde fuera de la clase:

```php
$o = new MiClase;
$valor = $o->miConst;  // error
$valor = MiClase::miConst;  //correcto
```

Y desde dentro, independientemente de si la constante es pública o no:

```php
$valor = $this->miConst;  // error
$valor = self::miConst;  // correcto
```

Para una lista de las constantes definidas, se usa `get_defined_constants()`. Para comprobar si una constante está definida, `defined()`.

Para acceder a su valor, simplemente se debe escribir su nombre, o usar `constant()`. La última forma tiene la ventaja que no hay que especificar el nombre de la constante en el código, ya que se le pasa un *string* con dicho nombre (se decide en tiempo de ejecución).

No se puede usar `const` dentro de un bloque condicional, pero sí `define()`.

## Constantes predefinidas

Existe una serie de constantes que están predefinidas. Por un lado, las *core predefined constants* están siempre disponibles. Dependiendo de las extensiones instaladas, habrá otras tantas disponibles.

Algunos ejemplos de constantes *core* pueden ser ***PHP_VERSION***, ***PHP_DEBUG***, ***PHP_EOL***, ***PHP_INT_MAX***, ***PHP_INT_MIN***, ***true***, ***false***, ***null***, etc.

## Constantes mágicas

Las *magic constants* pueden ir cambiando a lo largo de la ejecución, con lo que no son realmente constantes, aunque no se les puede asignar valor. Son *case insensitive*:

- ***\_\_LINE\_\_*** indica la línea del *script*.
- ***\_\_FILE\_\_*** es la ruta completa del archivo fuente (si es un archivo incluido, es la ruta del archivo incluido, no del que lo incluye).
- ***\_\_DIR\_\_*** es el directorio del archivo fuente. Equivale a `dirname(__FILE__)`.
- ***\_\_FUNCTION\_\_*** devuelve el nombre de la función actual. Si es una función anónima retorna ***\"\{closure}\"***.
- ***\_\_CLASS\_\_*** es el nombre de la clase actual, incluyendo el *namespace* en el que está definida (tipo ***\"Namesp\\Clase\"***). Si se usa en un *trait* devuelve el nombre de la clase que usa ese *trait*.
- ***\_\_TRAIT\_\_*** es el nombre del *trait* actual, incluyendo el *namespace* en el que está incluido (tipo ***\"Namesp\\Trait\"***).
- ***\_\_METHOD\_\_*** es el nombre del método actual.
- ***\_\_NAMESPACE\_\_*** es el nombre del *namespace* actual.
- ***ClassName::class*** es el nombre de la clase actual *fully qualified*.
