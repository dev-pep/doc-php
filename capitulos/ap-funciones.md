# Funciones más utilizadas

De entre la gran cantidad de funciones disponibles en *PHP*, veamos algunas de las más usadas.

## Arrays

```
array_diff(array $array, array ...$arrays): array
```

Compara ***array*** con el resto de *arrays*, y retorna un *array* con los valores de ***array*** que no están en el resto de *arrays*. Las claves de ***array*** se conservan tal cual.

```
array_filter(array $array, ?callable $callback = null, int $mode = 0): array
```

Itera sobre todos los **valores** del *array*, que va pasando a la función ***callback***, la cual, para cada elemento, retornará ***true*** para mantener el elemento en el *array* retornado (si no retorna verdadero, ese elemento queda fuera). Al final retorna un *array* filtrado.

Las claves se mantienen todas, y es posible que se produzcan *gaps* (para reindexar se puede usar `array_values()`).

Por defecto se pasan los valores a la ***callback***. Si ***mode*** es ***ARRAY_FILTER_USE_KEY***, se pasarán las claves. Si es ***ARRAY_FILTER_USE_BOTH*** se pasará valor y clave (en este orden).

```
array_key_exists(string|int $key, array $array): bool
```

Retorna ***true*** si la clave ***key*** existe en el *array* ***array***, o ***false*** si no.

```
array_key_first(array $array): int|string|null
```

Retorna la clave del primer elemento del *array*, sin afectar al apuntador interno.

```
array_keys(array $array, mixed $search_value, bool $strict = false): array
```

Retorna un *array* con las claves. Si se especifica uno o varios (*array*) valores en ***search_value*** (argumento opcional), solo las claves cuyos **valores** correspondientes coincidan con alguno de estos serán retornadas. La comparación no será estricta (`===`) por defecto, a no ser que indiquemos ***strict*** a ***true***.

```
array_map(?callable $callback, array $array, array ...$arrays): array
```

Retorna un *array* con los valores retornados por ***callback***. Esta función *callback* recibe un parámetro por cada *array* pasado por parámetro, y es llamada sucesivamente con todos los valores de los *arrays* de entrada. Por ello, normalmente los *arrays* pasados a `array_map()` suelen tener el mismo número de elementos.

```
array_merge(array ...$arrays): array
```

Retorna un *array* con los elementos de dos o más *arrays* mezclados. Las claves no se duplican, sino que se sobreescriben los elementos. Las claves numéricas son renumeradas. Si no queremos esta sobreescritura ni la renumeración se debe usar el operador `+` (si una clave existe en ambos *arrays* será ignorada la del segundo).

```
array_pop(array &$array): mixed
```

Elimina el último elemento del *array* y retorna ese elemento. El apuntador interno del *array* pasará a apuntar al primer elemento. Retorna ***null*** si el *array* está vacío.

```
array_push(array &$array, mixed ...$values): int
```

Añade los valores ***values*** al final del *array*, y retorna el nuevo número de elementos de este. Las claves numéricas serán renumeradas para empezar desde 0.

```
array_reverse(array $array, bool $preserve_keys = false): array
```

Retorna un *array* con los elementos de ***array*** con el orden invertido. Las claves numéricas son renumeradas, a no ser que ***preserve_keys*** sea ***true***.

```
array_search(mixed $needle, array $haystack,
    bool $strict = false): int|string|false
```

Busca un valor ***needle*** en un *array* ***haystack*** y retorna la clave del primer elemento coincidente. Si deseamos comparación estricta (`===`), ***strict*** debe ser verdadero. Si no se encuenta el elemento, retornará ***false***.

```
array_shift(array &$array): mixed
```

Elimina el primer elemento del *array* y retorna ese elemento. Las claves numéricas serán renumeradas para empezar desde 0. El apuntador interno del *array* pasará a apuntar al primer elemento. Retorna ***null*** si el *array* está vacío.

```
array_slice(array $array, int $offset, ?int $length = null, bool $preserve_keys = false): array
```

Retorna un *slice* de ***array***, empezando en la posición definida por ***offset*** y hasta un máximo de ***length*** elementos.

Si ***offset*** es negativo, la posición inicial se empieza a contar desde el final. Si ***length*** es negativo, se tomarán elementos hasta el que esté a esa distancia del último.

Por defecto se reordenarán y renumerarán los índices enteros, a no ser que ***preserve_keys*** sea verdadero. Los índices *string* se mantienen siempre igual.

```
array_unique(array $array, int $flags = SORT_STRING): array
```

Retorna un *array* sin elementos (valores) duplicados, a partir de ***array***. Las claves se mantienen. Si varios valores se comparan igual, se conservará la clave del primero de ellos. ***flags*** indica el modo de comparación, la cual es siempre estricta (`===`):

- ***SORT_REGULAR*** compara los elementos sin cambiar su tipo.
- ***SORT_NUMERIC*** compara numéricamente.
- ***SORT_STRING*** compara los elementos como *strings*.
- ***SORT_LOCALE_STRING*** compara los elementos como *strings* basándose en el locale actual.

```
array_unshift(array &$array, mixed ...$values): int
```

Añade los valores ***values*** al principio del *array*, y retorna el nuevo número de elementos de este. Las claves numéricas serán renumeradas para empezar desde 0.

```
array_values(array $array): array
```

Retorna un *array* con todos los **valores** del *array* original indexados numéricamente.

```
count(Countable|array $value, int $mode = COUNT_NORMAL): int
```

Retorna el número de elementos de ***value***. Si ***mode*** es ***COUNT_RECURSIVE*** o 1, contará también los elementos interiores en *arrays* multidimensionales.

```
current(array|object $array): mixed
```

Retorna el valor del elemento del *array* apuntado por el apuntador interno.

```
end(array|object &$array): mixed
```

Avanza el apuntador interno del *array* hasta el último elemento (se pasa por referencia porque modifica ese apuntador interno), y retorna su valor. Si está vacío retorna ***false***.

```
in_array(mixed $needle, array $haystack, bool $strict = false): bool
```

Retorna ***true*** si ***needle*** es igual (`==`) a alguno de los elementos en ***haystack***, o ***false*** en caso contrario. Los *strings* se comparan en modo *case-sensitive*. Si ***strict*** es ***true***, la comparación será estricta, es decir, incluyendo el tipo (`===`).

```
ksort(array &$array, int $flags = SORT_REGULAR): bool
```

Ordena *in-place* los elementos del *array* indicado. Si dos elementos se comparan igual, mantendrán su orden relativo. Establece el apuntador interno al primer elemento. Siempre retorna ***true***. En cuanto a los ***flags***, podemos destacar:

- ***SORT_REGULAR*** compara normalmente.
- ***SORT_NUMERIC*** compara numéricamente.
- ***SORT_STRING*** compara los elementos como *strings*.
- ***SORT_FLAG_CASE*** para comparaciones *case-insensitive*. Se puede combinar con ***SORT_STRING***.

```
next(array|object &$array): mixed
```

Es como `current()`, pero avanza el apuntador interno una posición (por eso se pasa por referencia) **antes** de retornar su valor. Si el apuntador estaba en la última posición, retornará ***false***.

```
prev(array|object &$array): mixed
```

Como `next()`, pero retrocediendo una posición en lugar de avanzar.

```
print(string $expression): int
```

Imprime la expresión pasada como argumento. Siempre retorna 1.

Es una construcción del lenguaje y no una función. No es necesario incluir los paréntesis de la lista de argumentos.

```
reset(array|object &$array): mixed
```

Establece el apuntador interno del *array* al primer elemento. Retorna el valor de ese elemento.

## Variables

```
echo(string $arg1, string $... = ?): void
```

Imprime las expresiones indicadas. No es una función sino una construcción del lenguaje, y no es necesario incluir los paréntesis de la lista de argumentos.

Posee la sintaxis abreviada `<?= $expresion ?>`.

```
empty(mixed $var): bool
```

Retorna ***true*** si ***var*** no tiene valor asignado, o tiene un valor falso (***false***, cero, *string* vacío, etc.). De lo contrario retornará ***false***.

```
gettype(mixed $value): string
```

Retorna un *string* descriptivo del tipo de ***value***.

```
intval(mixed $value, int $base = 10): int
```

Retorna el valor entero correspondiente al argumento (escalar, normalmente *string*) indicado, usando la base indicada para la conversión. Si la base es 0, se elegirá según el formato del *string*. Si hay error, retornará 0.

```
is_array(mixed $value): bool
```

Retorna ***true*** si la variable es un *array* y ***false*** en caso contrario.

```
is_bool(mixed $value): bool
```

Retorna ***true*** si la variable es booleana y ***false*** en caso contrario.

```
is_callable(mixed $value, bool $syntax_only = false, string &$callable_name = null): bool
```

Retorna ***true*** si ***value*** es un *callable* (nombre de función, objeto *callable*). Si ***syntax_only*** es verdadero, solo comprueba si ***value*** se refiere a una función o método, y no a ningún otro tipo de *callable*. Si se le proporciona ***callable_name***, recibirá el nombre del *callable*.

```
is_int(mixed $value): bool
```

Retorna ***true*** si la variable es un entero y ***false*** en caso contrario.

```
is_null(mixed $value): bool
```

Retorna ***true*** si la variable tiene valor ***null*** y ***false*** en caso contrario.

```
is_numeric(mixed $value): bool
```

Retorna ***true*** si la variable tiene valor numérico o *string* numérico y ***false*** en caso contrario.

```
isset(mixed $var, mixed ...$vars): bool
```

Comprueba si una o más variables han sido establecidas. Una variable está establecida (***set***) si tiene un valor asignado y es distinto de ***null***. La función retornará ***true*** si todas las variables pasadas como argumento están establecidas, y ***false*** en caso contrario.

Véase `unset()`.

```
print_r(mixed $value, bool $return = false): string|bool
```

Imprime el contenido de ***value*** de una forma legible para un humano. Si ***return*** es verdadero, en lugar de imprimir la información la retornará. De lo contrario, retornará ***true***.

```
serialize(mixed $value): string
```

Retorna un *string* que representa tanto los valores como los tipos de un objeto ***value***. Se trata de un *stream* de *bytes*, no de un *string* codificado en algún formato *multibyte*. Por lo tanto hay que tener en cuenta que es un *stream* binario, y tratarlo como tal (por ejemplo, en una base de datos habrá que almacenarlo en un *BLOB* y no en un *CHAR* o *TEXT*).

Para un formato más seguro, se debe utilizar `json_encode()` y `json_decode()`.

```
unserialize(string $data, array $options = []): mixed
```

Retorna un objeto a partir del objeto serializado con `serialize()`.

***options*** es un *array* de opciones que admite, en una clave ***allowed_classes***, uno de estos tres valores:

- ***true*** indica que se aceptan todas las clases (por defecto).
- ***false*** indica que no se aceptan clases.
- Un *array* conteniendo los nombres de las clases aceptadas.

```
unset(mixed $var, mixed ...$vars): void
```

Desestablece (*unsets*) las variables pasadas como parámetro.

```
var_dump(mixed $value, mixed ...$values): void
```

Imprime el contenido de las expresiones que se le pasan como argumento.

## Strings

```
chr(int $codepoint): string
```

Retorna un *string* de un solo carácter cuyo código *ASCII* (0-255) está definido por ***codepoint***.

```
explode(string $separator, string $string, int $limit = PHP_INT_MAX): array
```

Retorna un *array* formado por *substrings* de ***string*** delimitados por el *string* ***separator***. El *array* no tendrá más de ***limit*** elementos (si ***limit*** es negativo, se retornarán todos los elementos excepto los últimos ***-limit***).

```
htmlspecialchars(string $string,
    int $flags = ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401,
    ?string $encoding = null,
    bool $double_encode = true): string
```

Retorna un *string* en el que los caracteres especiales son convertidos a entidades *HTML*.

***flags*** es una combinación de valores, entre los que podemos mencionar estos:

- ***NT_COMPAT*** convierte comillas dobles pero no simples.
- ***ENT_QUOTES*** convierte todas las comillas.
- ***ENT_NOQUOTES*** no convierte ningún tipo de comillas.
- ***ENT_IGNORE*** descarta secuencias iválidas, en lugar de retornar un *string* vacío. No recomendado.
- ***ENT_SUBSTITUTE*** reemplaza secuencias inválidas con el carácter *Unicode* ***U+FFFD*** (*UTF-8*) o `&#xFFFD;` (no *UTF-8*) en lugar de retornar un *string* vacío.
- ***ENT_HTML401*** trata el código como *HTML* 4.01.
- ***ENT_HTML5*** trata el código como *HTML* 5.

Adicionalmente se puede definir la codificación del *string* en ***encoding***: ***ISO-8859-1*** (europeo occidental *Latin-1*), ***ISO-8859-15*** (europeo occidental *Latin-9*, añade algunos símbolos como el euro), ***UTF-8***, ***cp1252*** (europeo occidental de *Windows*), etc.

Si ***double_encode*** es ***false***, las entidades *HTML* ya existentes no se recodificarán.

```
htmlspecialchars_decode(string $string,
    int $flags = ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML401): string
```

Es el inverso de `htmlspecialchars()`.

```
implode(string $separator, array $array): string
```

Retorna un *string* con la concatenación de los *strings* de ***array***, intercalados con ***separator*** (por defecto *string* vacío si solo se pasa un argumento).

```
is_string(mixed $value): bool
```

Retorna ***true*** si el valor es un *string* o ***false*** si no.

```
ltrim(string $string, string $characters = " \n\r\t\v\x00"): string
```

Es como `trim()` pero solo al principio del *string*.

```
md5(string $string, bool $binary = false): string
```

Retorna un *hash* *MD5* del *string* indicado. Si ***binary*** es ***true***, el *string* retornado será en formato binario.

```
ord(string $character): int
```

Retorna el código *ASCII* (0-255) del primer carácter de ***character***. Si el *string* tiene una codificación *single-byte* (*ASCII*, *ISO-8859* o *Windows 1252*, por ejemplo) la función retornará simplemente la posición del carácter dentro del juego de caracteres. Pero si tiene una codificación *multibyte* (como *UTF-8* o *UTF-16*), retornará simplemente el primer *byte* de la secuencia.

```
rtrim(string $string, string $characters = " \n\r\t\v\x00"): string
```

Es como `trim()` pero solo al final del *string*.

```
sprintf(string $format, mixed ...$values): string
```

Retorna un *string* formateado a partir de un *string* de formato ***format*** y una serie indefinida de valores, de forma análoga a la función `sprintf()` de *C*.

```
strlen(string $string): int
```

Retorna la longitud del *string*.

```
strpos(string $haystack, string $needle, int $offset = 0): int|false
```

Retorna la posición que ocupa ***needle*** dentro de ***haystack***, o ***false*** si no existe. I indicamos un valor positivo en ***offset***, la búsqueda se iniciará en esa posición (si es negativo, esa posición se cuenta desde el final del *string*).

```
str_repeat(string $string, int $times): string
```

Retorna un *string* consistente en el *string* de entrada repetido ***times*** veces.

```
str_replace(array|string $search,
    array|string $replace, string|array $subject,
    int &$count = null): string|array
```

Retorna un *string* en el que todas las ocurrencias de ***search*** en ***subject*** son sustituidas por ***replace***. Si ***search*** y ***replace*** son *arrays*, cada elemento de ***search*** se sustituirá por el elemento correspondiente en ***replace*** (si ***replace*** tiene menos elementos que ***search***, los elementos de ***search*** sin correspondencia se sustituirán por un *string* vacío). Si ***search*** es un *array* y ***replace*** es un *string*, se sustituirán todos los elementos de ***search*** por ese *string*. Si ***subject*** es un *array* (de *strings*), la sustitución se producirá en todos sus elementos, y el resultado retornado será un *array* con todos esos *strings* sustituidos. Si se especifica ***count***, este recibirá el número de sustituciones realizadas.

```
strrpos(string $haystack, string $needle, int $offset = 0): int|false
```

Retorna la posición en la que ***needle*** aparece por última vez dentro de ***haystack***.

Si ***offset*** no es negativo, se realiza la búsqueda a partir de esa posición, hacia la derecha. Si es negativo, se empieza la búsqueda saltándose esa cantidad de caracteres desde el final, y se sigue la búsqueda hacia la izquierda, esta vez buscando la primera ocurrencia.

Si no hay coincidencia, la función retorna ***false***.

```
strtolower(string $string): string
```

Retorna el *string* convertido a minúsculas.

```
strtoupper(string $string): string
```

Retorna el *string* convertido a mayúsculas.

```
strtr(string $str, string $from, string $to): string
```

```
strtr(string $str, array $replace_pairs): string
```

Retorna un *string* con caracteres o *substrings* de ***str*** reemplazados.

Si se dan tres argumentos, cada vez que un carácter de ***str*** coincida con un carácter (*single-byte*) en ***from***, será reemplazado por el correspondiente carácter en ***to***. Si ***from*** y ***to*** tienen un número distinto de caracteres, los caracteres extra serán ignorados.

Si se dan dos argumentos, ***replace_pairs*** será un *array* cuyas claves serán *substrings* a sustituir en ***str*** por su valor asociado.

```
substr(string $string, int $start, int $length = ?): string
```

Retorna un *substring* de ***string***, iniciando en la posición ***offset***, y con un máximo de ***length*** caracteres (por defecto hasta el final del *string*). Si ***offset*** es negativo, se contará desde el final. Si ***length*** es negativo, indica la cantidad de caracteres a omitir del final del *string*.

```
trim(string $string, string $characters = " \n\r\t\v\x00"): string
```

Retorna un *string* con los caracteres en blanco eliminados del principio y el final. Si queremos definir nuestros propios caracteres a eliminar, los especificaremos en ***characters***.

```
ucfirst(string $string): string
```

Retorna el *string* de entrada con el primer carácter en mayúsculas.

```
utf8_encode(string $string): string
```

Convierte un *string* de codificación *ISO-8859-1* a *UTF-8*. La codificación *ISO-8859-1*, llamada *Latin 1* es muy similar a la codificación *Windows-1252*, y es utilizada frecuentemente por páginas y servidores *web*.

## Expresiones regulares

Las expresiones regulares deben ir entre barras: ***/.../***

```
preg_match(string $pattern, string $subject,
    array &$matches = null, int $flags = 0,
    int $offset = 0): int|false
```

Busca una coincidencia de la expresión regular ***pattern*** dentro de ***subject***. Si se indica ***matches*** recibirá un *array* en el cual el elemento 0 es el texto completo que ha coincidido con el patrón, el 1 el primer grupo capturado entre paréntesis, el 2 el segundo, etc.

***flags*** puede ser una combinación de ***PREG_OFFSET_CAPTURE*** (en lugar de *strings*, los elementos de ***matches*** serán un *array* con dos valores: el *string* en sí y el *offset* dentro de ***subject***) y ***PREG_UNMATCHED_AS_NULL*** (los subpatrones no coincidentes darán ***null*** en lugar de un *string* vacío).

***offset*** indica el punto de ***subject*** donde empezará la búsqueda.

La función retorna 1 si ha habido coincidencia, 0 si no, o ***false*** si ha habido error.

```
preg_match_all(string $pattern,
    string $subject, array &$matches = null,
    int $flags = 0, int $offset = 0): int|false
```

Es como `preg_match()`, pero en este caso se buscan **todas** las coincidencias posibles del patrón dentro del *string*. Tras una coincidencia, la siguiente se empieza desde el final de la anterior dentro del *string*. En este caso, hay dos valores más que pueden combinarse en ***flags*** (aunque estos dos no tienen sentido juntos):

- ***PREG_PATTERN_ORDER***: el primer elemento de ***matches*** es un *array* con los *full matches*, ***matches[1]*** son todos los *matches* del primer subpatrón entre paréntesis, etc.
- ***PREG_SET_ORDER***: el primer elemento de ***matches*** es un *array* con el primer conjunto de coincidencias, el segundo un *array* con el segundo conjunto de coincidencias, etc.

```
preg_replace(mixed $pattern,
    mixed $replacement, mixed $subject,
    int $limit = -1, int &$count = ?): mixed
```

Busca en ***subject*** coincidencias de ***pattern*** y las reemplaza con ***replacement***. ***limit*** indica el número máximo de sustituciones (por defecto, sin límite). Si se especifica ***count***, recibirá el número de sustituciones realizadas.

***pattern*** puede ser un *array* de *strings*, en cuyo caso cada elemento buscará una correspondencia en ***replacement*** de la misma forma que en el caso de `str_replace()`.

Si ***subject*** es un *array* de *strings*, las sustituciones se realizarán en cada uno de ellos, y el resultado retornado será a su vez un *array* con los *strings* sustituidos.

```
preg_split(string $pattern, string $subject,
    int $limit = -1, int $flags = 0): array|false
```

Trocea el *string* ***subject*** según el delimitador definido por la expresión regular ***pattern***. El resultado devuelto es un *array* con todos los fragmentos. ***limit*** indica el número máximo de *substrings* retornados (el último de ellos es el resto del *string*). Un límite de 0 o -1 indica que no hay límite.

***flags*** puede ser una combinación de:

- ***PREG_SPLIT_NO_EMPTY***: no se incluyen fragmentos vacíos.
- ***PREG_SPLIT_DELIM_CAPTURE***: los subpatrones entre paréntesis en el delimitador serán capturados e incluidos en el resultado retornado.
- ***PREG_SPLIT_OFFSET_CAPTURE***: se retornarán también los *offsets* de los fragmentos.

## Sistema de archivos

Las rutas relativas se evaluarán en relación al directorio actual. En sistemas *Windows* se pueden indicar tanto barras como barras invertidas para crear las rutas.

> Dependiendo del sistema operativo, los nombres de archivos son *case-sensitive* (*Unix*) o no (*Windows*).

```
basename(string $path, string $suffix = ""): string
```

Retorna el componente más a la derecha de una ruta. Si el componente termina en ***suffix***, este será descartado del resultado devuelto.

```
dirname(string $path, int $levels = 1): string
```

Retorna la ruta del directorio padre de la ruta indicada. Es posible subir más de un nivel mediante ***levels***.

```
fclose(resource $stream): bool
```

Cierra el *stream* ***stream***. Retorna ***true*** si tiene éxito, y ***false*** en caso contrario.

```
file_exists(string $filename): bool
```

Retorna ***true*** si el archivo cuya ruta es ***filename*** existe, o ***false*** en caso contrario.

```
file_get_contents(string $filename,
    bool $use_include_path = false,
    ?resource $context = null, int $offset = 0,
    ?int $length = null): string|false
```

Es la manera recomendada de leer el contenido de un archivo.

***use_include_path*** indica si se debe buscar en ciertos directorios definidos en la configuración. ***context*** suele ser ***null***, a no ser que deseemos pasarle un contexto creado con `stream_context_create()`.

***offset*** indica dónde empezar la lectura del archivo. Si es negativo, se cuenta desde el final del mismo. No funciona en *filesystems* remotos. ***length*** es el máximo número de *bytes* a leer.

```
file_put_contents(string $filename,
    mixed $data, int $flags = 0,
    ?resource $context = null): int|false
```

Escribe ***data*** (*string* o *array* unidimensional) en el archivo ***filename***. ***context*** suele ser ***null***, a no ser que deseemos pasarle un contexto creado con `stream_context_create()`.

Si el archivo no existe se creará.

Los ***flags*** pueden combinar (con or binario):

- ***FILE_USE_INCLUDE_PATH*** busca el archivo en el directorio *include* configurado.
- ***FILE_APPEND*** añade los datos al archivo si ya existe, en lugar de sobreescribirlo.
- ***LOCK_EX*** adquiere un bloqueo exclusivo del archivo mientras escribe.

Retorna el número de *bytes* escritos o ***false*** si hay error.

```
fopen(string $filename, string $mode,
    bool $use_include_path = false,
    ?resource $context = null): resource|false
```

Abre un archivo (ruta) o *URL* (schema://...) definido en ***filename***.

Al abrir, si no se especifica lo contrario, el apuntador de lectura/escritura se sitúa al principio del *stream*.

El modo de apertura se indica en ***mode***: algunos modos son ***r*** (*read only*), ***r+*** (lectura y escritura), ***w*** (escritura, con creación o truncado), ***w+*** (como ***w***, pero también permite lectura), ***a*** (como ***w*** pero situando el apuntador al final, `fseek()` no tiene efecto al abrir en este modo), ***a+*** (como ***a*** pero también permite lectura, `fseek()` solo afecta a la lectura).

***use_include_path*** indica si se debe buscar en ciertos directorios definidos en la configuración. ***context*** suele ser ***null***, a no ser que deseemos pasarle un contexto creado con `stream_context_create()`.

La función retorna un recurso archivo, o ***false*** si hay error.

```
fwrite(resource $stream, string $data, ?int $length = null): int|false
```

Es la manera recomendada de escribir en un archivo.

Escribe ***data*** en ***stream***, como máximo ***length*** *bytes* (si definimos ese valor). Retorna la cantidad de *bytes* escritos o ***false*** si hay error.

```
is_dir(string $filename): bool
```

Retorna ***true*** si la ruta especificada corresponde a un directorio existente, y ***false*** en caso contrario.

```
is_file(string $filename): bool
```

Retorna ***true*** si la ruta especificada corresponde a un archivo existente, y ***false*** en caso contrario.

```
mkdir(string $directory, int $permissions = 0777,
    bool $recursive = false,
    ?resource $context = null): bool
```

Intenta crear el directorio ***directory***, con los permisos indicados en ***permissions*** (no funciona en *Windows*). Si ***recursive*** es verdadero, se crearán los directorios necesarios de la ruta. Retorna ***true*** si tiene éxito y ***false*** en caso contrario.

```
realpath(string $path): string|false
```

Retorna la ruta especificada, resolviendo nombres como ***.***, ***..*** o enlaces simbólicos. La posible barra final es descartada. Si hay error retorna ***false***.

```
unlink(string $filename, ?resource $context = null): bool
```

Elimina el archivo ***filename***. Retorna ***true*** si tiene éxito y ***false*** en caso contrario.

## Funciones

```
call_user_func(callable $callback, mixed ...$args): mixed
```

Invoca la función ***callback***, pasándole los argumentos indicados (cero o más, que se pasarán siempre por valor a la *callback*). Retorna el valor que retorna la *callback*, o ***false*** si hay error.

```
call_user_func_array(callable $callback, array $args): mixed
```

Es como `call_user_func()`, pero los argumentos se indican en un *array* ***args***.

```
func_get_arg(int $position): mixed
```

Retorna el argumento de la función actual que ocupa la posición ***position*** (o ***false*** si hay error).

```
func_get_args(): array
```

Retorna un *array* con los argumentos de la función actual.

```
function_exists(string $function): bool
```

Retorna ***true*** si existe la función con nombre ***function***, o ***false*** en caso contrario.

```
func_num_args(): int
```

Retorna el número de argumentos pasados a la función actual.

## Clases y objetos

```
class_exists(string $class, bool $autoload = true): bool
```

Retorna ***true*** si la clase indicada existe, o ***false*** en caso contrario. Por defecto se autocarga la clase si es necesario (y está disponible la autocarga para esa clase), a no ser que indiquemos ***autoload*** como ***false***.

El nombre de la clase debemos pasárselo *fully-qualified*.

```
get_class(object $object = ?): string
```

Retorna el nombre de la clase (*fully qualified*, equivalente a `<Clase>::class`) del objeto indicado. Dentro de una clase se puede omitir el argumento, en cuyo caso se retornará el nombre de esa clase.

```
is_object(mixed $value): bool
```

Retorna ***true*** si el valor es un objeto, o ***false*** en caso contrario.

```
method_exists(object|string $object_or_class, string $method): bool
```

Retorna ***true*** si existe un método con nombre ***method*** dentro del objeto o nombre de clase indicado en ***object_or_class***. De lo contrario, retorna ***false***.

## Fecha y hora

```
date(string $format, ?int $timestamp = null): string
```

Retorna un *string* correspondiente a ***timestamp*** (por defecto el momento actual), formateado según ***format***. Si ***timestamp*** no es numérico, la función retorna ***false***.

Ejemplos de algunos componentes del formato:

- ***d***: día del mes, de 01 a 31.
- ***j***: día del mes, de 1 a 31.
- ***N***: día de la semana ISO 8601, de 1 (lunes) a 7 (domingo).
- ***m***: mes, de 01 a 12.
- ***n***: mes, de 1 a 12.
- ***t***: número de días en el mes de la fecha.
- ***Y***: año en 4 dígitos.
- ***y***: año en 2 dígitos.
- ***H***: hora, de 00 a 23.
- ***i***: minutos, de 00 a 59.
- ***s***: segundos, de 00 a 59.

Los caracteres no correspondientes a un elemento de formato se escribirán tal cual.

```
microtime(bool $as_float = false): string|float
```

Retorna el *timestamp* actual en microsegundos (no disponible en todos los sistemas operativos). Si ***as_float*** es verdadero, en lugar de retornar un *string* se retornará un *float*.

```
strtotime(string $datetime, ?int $baseTimestamp = null): int|false
```

Convierte un *string* que define una fecha (en varios formatos ingleses posibles) a un *timestamp*, que será relativo a ***baseTimestamp***, o al momento actual si no se indica este. Retorna ese *timestamp* relativo, o ***false*** si hay error.

```
time(): int
```

Retorna *timestamp* actual, es decir, el número de segundos desde la época *Unix* (01/01/1970, 00:00:00 *GMT*).

## JSON

```
json_decode(string $json, ?bool $associative = null,
    int $depth = 512, int $flags = 0): mixed
```

Retorna un objeto construido a partir del *string JSON* ***json***. Solo para *strings* codificados mediante *UTF-8*. La profundidad máxima a considerar se define en ***depth***. Si ***associative*** es ***true***, el resultado será un *array* asociativo. Si es ***false*** será un objeto. Si es ***null*** (por defecto), dependerá del *flag* ***JSON_OBJECT_AS_ARRAY***.

Si no se puede construir el resultado o el *string* tiene una profundidad superior a la indicada, la función retorna ***null***.

```
json_encode(mixed $value, int $flags = 0,
    int $depth = 512): string|false
```

Retorna la representación *JSON* del objeto ***value***. La profundida máxima a utilizar (***depth***) debe ser mayor que 0. La función retorna ***false*** si hay error.

## Red

```
header(string $header, bool $replace = true, int $response_code = 0): void
```

Envía un encabezado *HTTP*. Debe ser lo primero que se haga al gestionar la *request* actual, ya que de lo contrario, si se envía cualquier tipo de salida (incluyendo un simple espacio en blanco), ya no se enviarán encabezados al cliente, con lo que la función no tendrá efecto.

***header*** incluye el nombre del encabezado y su valor, del tipo ***Nombre: valor***.

Existen dos casos especiales. Si el *string* empieza por ***HTTP***, se usará para establecer el código de retorno al navegador:

```php
header("HTTP/1.1 404 Not Found");
```

El otro es el envío del encabezado específico ***Location***, que de hecho retorna ese encabezado y además retorna un código de redirección 302 (si no hemos establecido el código de retorno como se ha visto antes).

```php
header("Location: https://pepe.com");
exit;  // no se debe ejecutar más código al redirigir
```

***replace*** indica si el encabezado debe reemplazar a un encabezado anterior con el mismo nombre (por defecto), o se admiten varios encabezados con ese mismo nombre. ***response_code*** especifica el código de respuesta (siempre que ***header*** no esté vacío).

## Control de salida

```
ob_clean(): bool
```

Es como `ob_end_clean()`, pero no desactiva el *buffering* de salida.

```
ob_get_clean(): string|false
```

Retorna el contenido del *buffer* de salida, tras limpiarlo y desactivar el *buffering* de salida. Si dicho *buffering* no estaba activo, retorna ***false***.

```
ob_get_contents(): string|false
```

Retorna el contenido del *buffer* de salida. Si el *buffering* de salida no está activo, retorna ***false***.

```
ob_get_flush(): string|false
```

Es como `ob_end_flush()` pero el valor de retorno es el contenido del *buffer* (o ***false*** si no estaba activo el *buffering* de salida).

```
ob_end_clean(): bool
```

Descarta el contenido del *buffer* de salida (si lo hay), y desactiva el *buffering* de salida. Retorna ***true*** si tiene éxito y ***false*** en caso contrario (por ejemplo cuando el *buffering* de salida no está activo).

```
ob_end_flush(): bool
```

Envía a la salida el contenido del *buffer* de salida (si lo hay), limpia el *buffer* y desactiva el *buffering* de salida. Retorna ***true*** si tiene éxito y ***false*** en caso contrario (por ejemplo cuando el *buffering* de salida no está activo).

```
ob_flush(): bool
```

Es como `ob_end_flush()`, pero no desactiva el *buffering* de salida.

```
ob_start(callable $callback = null, int $chunk_size = 0,
    int $flags = PHP_OUTPUT_HANDLER_STDFLAGS): bool
```

A partir de su invocación, la salida producida a través de cualquiera de las funciones que la producen se va insertando en un *buffer*. A partir de ese momento, el *script* no envía ninguna salida (solo encabezados).

Es posible anidar varios *buffers* llamando varias veces a esta función. Es importante terminar cada uno de los *buffers*. Si el *script* termina y sigue habiendo *buffers* activos, estos son *flushed* automáticamente.

***callback*** es una función opcional que recibe un *string* (contenido a ser *flushed*) y retorna un *string* (contenido modificado que será *flushed*). Como segundo parámetro, opcional, esta función recibe los *flags* del *buffer* modificado (por si deben ser distintos al original). Si la función retorna ***false***, el *buffer* original es enviado. Desde la *callback* no se pueden llamar las funciones `ob_end_clean()`, `ob_end_flush()`, `ob_clean()`, `ob_flush()` y `ob_start()`, y tampoco funciones como `print_r()`.

Si se especifica ***chunk_size***, se hará *flush* del *buffer* automáticamente cada vez que su tamaño llegue o exceda tal número. Un tamaño 0 indica que no habrá *flush* automático.

***flags*** es una combinación de valores que permiten la limpieza, *flush* y/o desactivación (eliminación) del *buffer*: ***PHP_OUTPUT_HANDLER_CLEANABLE***, ***PHP_OUTPUT_HANDLER_FLUSHABLE*** y ***PHP_OUTPUT_HANDLER_REMOVABLE***. Si se quieren activar los tres, se puede simplemente indicar ***PHP_OUTPUT_HANDLER_STDFLAGS***.

Esta función retorna ***true*** si tiene éxito, o ***false*** en caso contrario.

## Matemáticas

```
ceil(int|float $num): float
```

Retorna un número entero, redondeando al alza si hace falta, aunque el tipo retornado será ***float***.

```
floor(int|float $num): float
```

Retorna un número entero, redondeando a la baja si hace falta, aunque el tipo retornado será ***float***.

```
max(array $values): mixed
```

```
max(mixed $value1, mixed $value2, mixed $... = ?): mixed
```

Retorna el mayor (numéricamente) de los valores. Si solo se le pasa un *array*, retornará el mayor de los valores de este.

```
min(array $values): mixed
```

```
min(mixed $value1, mixed $value2, mixed $... = ?): mixed
```

Retorna el menor (numéricamente) de los valores. Si solo se le pasa un *array*, retornará el menor de los valores de este.

```
round(int|float $num, int $precision = 0, int $mode = PHP_ROUND_HALF_UP): float
```

Retorna el valor flotante redondeado de ***num*** a la precisión (decimales) ***precision***. Si la precisión es negativa, se referirá a posiciones **a la izquierda** del punto decimal.

***mode*** puede ser uno de los siguientes:

- ***PHP_ROUND_HALF_UP*** redondea alejándose del 0.
- ***PHP_ROUND_HALF_DOWN*** redondea hacia el 0.
- ***PHP_ROUND_HALF_EVEN*** redondea hacia el valor par más próximo.
- ***PHP_ROUND_HALF_ODD*** redondea hacia el valor impar más próximo.

## URLs

```
base64_decode(string $string, bool $strict = false): string|false
```

Decodifica datos codificados mediante *MIME base64* y retorna el *string* original.

Si ***strict*** es ***true***, si la entrada contiene caracteres fuera del alfabeto *base64*, la función retornará ***false***. De lo contrario, estos caracteres serán simplemente descartados.

```
base64_encode(string $string): string
```

Codifica los datos usando *MIME base64* (reversible). Retorna tal codificación.

Se utiliza sobre todo para transporte de datos binarios.

```
parse_url(string $url, int $component = -1): int|string|array|null|false
```

Retorna un *array* asociativo con todas las partes de la *URL* indicada. Si solo se desea obtener uno de los elementos de la *URL*, indicaremos en ***component*** uno de estos valores, en cuyo caso se retornará un *string*: ***PHP_URL_SCHEME***, ***PHP_URL_HOST***, ***PHP_URL_USER***, ***PHP_URL_PASS***, ***PHP_URL_PATH***, ***PHP_URL_QUERY*** o ***PHP_URL_FRAGMENT***. Si indicamos ***PHP_URL_PORT*** se retornará un entero. Si el componente solicitado no existe, se retornará ***null*** (no es lo mismo un componente vacío que un componente inexistente).

Si la *URL* tiene un formato incorrecto, la función retornará ***false***.

```
urldecode(string $string): string
```

Decodifica una *URL* al contrario de `urlencode()`.

```
urlencode(string $string): string
```

Retorna un *string* con el *URL* codificado de tal modo que todos los caracteres no alfanuméricos (excepto guión bajo ***\_***) se sustituyen por un signo de porcentaje (***%***) seguido por dos dígitos hexadecimales, y los espacios se sustituyen por signos de suma (***+***).

## Información *PHP*

```
extension_loaded(string $extension): bool
```

Retorna ***true*** si la extensión *PHP* indicada está cargada, o ***false*** si no.

```
ini_get(string $option): string|false
```

Retorna el valor de la opción de configuración *PHP* indicada, como *string*. Los valores ***null*** retornan un *string* vacío. Si la opción no existe, retorna ***false***.

```
ini_set(string $option, string|int|float|bool|null $value): string|false
```

Establece la opción de configuración *PHP* indicada al valor de ***value***. Se mantendrá dicho valor durante la ejecución del *script*, tras lo cual se restaurará. No todas las opciones pueden cambiarse con esta función.

Si tiene éxito, retorna el valor antiguo como *string*. De lo contrario retorna ***false***.

```
version_compare(string $version1, string $version2,
    ?string $operator = null): int|bool
```

Compara *strings* con versiones de *PHP* con su formato estándar.

Por defecto, la función retorna -1 si ***version1*** es inferior a ***version2***, 0 si son iguales o 1 si es superior. En cambio, si especificamos ***operator***, retornará ***true*** si la comparación se cumple, y ***false*** en caso contrario. Posibles valores de ***operator*** son `<`, `lt`, `<=`, `le`, `>`, `gt`, `>=`, `ge`, `==`, `=`, `eq`, `!=`, `<>` o `ne`.

## Errores

```
error_reporting(?int $error_level = null): int
```

Establece los tipos de error que producirán un mensaje de error en la salida. Establece en *runtime* el valor de la directiva de configuración ***error_reporting***. Si no le pasamos ***error_level***, la función simplemente retornará la configuración actual. El valor ***E_ALL*** reporta todo tipo de errores y avisos.

## iconv

```
iconv(string $from_encoding, string $to_encoding, string $string): string|false
```

Retorna un *string* resultante de convertir ***string*** a una codificación distinta. ***from_encoding*** es la codificación del *string* origen, y ***to_encoding*** es la codificación final que deseamos.

Si hay error, retorna ***false***.

Ejemplos de codificaciones son ***ISO-8859-1*** (europeo occidental *Latin-1*), ***ISO-8859-15*** (europeo occidental *Latin-9*, que añade algunos símbolos como el euro), ***UTF-8***, ***CP1252*** (europeo occidental de *Windows*, alias de ***Windows-1252***, y muy similar a ***ISO-8859-1***), etc.

## Varias

```
defined(string $constant_name): bool
```

Retorna ***true*** si la **constante** con el nombre indicado existe, o ***false*** en caso contrario.

```
exit(string $status = ?): void
```

```
exit(int $status): void
```

Imprime un mensaje (si ***status*** es un *string*) y termina la ejecución. Si ***status*** es un entero, se utilizará como el estado de salida y no será imprimido. El código 0 indica éxito. Pueden usarse también los códigos 1-254.

Se trata de una construcción del lenguaje y no de una función, y puede indicarse sin paréntesis si no le pasamos ningún argumento.

La construcción `die()` (o `die`) es equivalente.

```
uniqid(string $prefix = "", bool $more_entropy = false): string
```

Genera y retorna un ID único en un *string* basado en la hora actual en microsegundos. Sin embargo, es posible que el valor generado no sea único. Para aumentar las probabilidades de que este sea único deberemos establecer  ***more_entropy*** a ***true***.

Podemos indicar un prefijo para este ID mediante ***prefix***. Suponiendo un prefijo vacío, el *string* retornado tendrá 13 caracteres. Con ***more_entropy***, 23.
