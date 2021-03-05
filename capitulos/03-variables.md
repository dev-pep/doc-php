# Variables

Las variables en *PHP* empiezan por el signo ***$*** y siguen con una secuencia de caracteres *ASCII*, de los cuales, el primero no puede ser numérico. Son *case-sensitive*. La variable ***$this*** no puede asignarse.

Las variables no son referencias (como en *Python*), con lo que asignar una variable a otra resultará en dos variables completamente desvinculadas. Sin embargo sí se puede asignar "por referencia" (como en *Python*):

```php
$a = &$b;
```

Para saber si una variable ha sido inicializada se usa `isset()`.

## Ámbito (*scope*)

El *scope* de la variable empieza donde está declarada, hacia adelante (incluyendo archivos incluidos y requeridos). Dentro de una función, tendrá *scope* local, por lo que no se podrá acceder a una variable de un ámbito exterior con el mismo nombre. Si queremos acceder a variables globales, se declararán con `global`.

Otra forma de acceder a las variables globales es mediante el *array* ***$GLOBALS***, usando como clave el nombre de la variable global deseada. Este *array* es **superglobal** (accesible en **todos** los *scopes*).

Una variable estática (`static`) existe dentro de una función únicamente, pero no pierde su valor entre llamadas. Se deben inicializar, y solo son inicializadas la primera vez.

## Variables de nombre variable (*variable variables*)

El código:

```php
$a = 'hello';
$hello = 'world';
```

Es equivalente a:

```php
$a = 'hello';
$$a = 'world';
```

En *arrays*: `$$arr[0]` es ambiguo, con lo que debemos especificar `${$arr[0]}` o `${$arr}[0]`.

Se pueden incluir tantos niveles como se quiera: ***$$$a***, ***$$$$a***, etc.

## Variables de fuentes externas

### Formularios HTML

Al recoger el resultado de un formulario *HTML* se puede obtener el valor de los distintos controles mediante ***$_POST['nombre']*** o ***$_GET['nombre']***, según el método del *request* sea *post* o *get*, y donde ***'nombre'*** es un string con el nombre (atributo ***name***) del control. Mediante ***$_GET*** se puede obtener el valor de los distintos parámetros de la *query string*. Los puntos y espacios en el nombre de un control se convierten en guión bajo (***_***) para acceso desde ***PHP***.

Cuando el *submit* se hace a través de una imagen, el navegador incluye las coordenadas del clic en dos variables ***sub.x*** y ***sub.y***, que deberán tratarse en ***PHP*** como ***sub_x*** y ***sub_y***.

### Cookies

*PHP* trata las *cookies* de forma transparente. Si queremos enviar una *cookie* al navegador (recordemos que estamos en el servidor), debemos hacerlo **antes** de producir cualquier tipo de salida, es decir antes de la etiqueta `<html>`, ya que las *cookies* van en las cabeceras *HTTP*. Para enviar una *cookie* se usa la función `setcookie()`. Una vez la *cookie* está creada, sus datos serán enviados al servidor en cada *request* y serán accesibles a través de los *arrays* ***$_COOKIE*** y ***$_REQUEST***.

Los valores leídos quedarán como *strings* y tendrán que ser tratados y convertidos según necesidades.
