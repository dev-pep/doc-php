# SQL

Los comandos terminan en punto y coma (***;***), y son *case-insensitive*. Se suelen escribir en mayúsculas para diferenciarlos de los elementos de las bases de datos (tablas, campos, etc.), también *case-insensitive*, que se suelen indicar en minúsculas.

## Tipos de datos

Nos centraremos en los tipos de datos de *MySQL*/*MariaDB*, aunque no debería cambiar significativamente en *Microsoft SQL Server*.

### Strings

- ***CHAR(size)***: *string* de longitud fija. Tamaño de 0 a 255 caracteres (por defecto 1).
- ***VARCHAR(size)***: *string* de longitud variable. Tamaño (máximo) de 0 a 65.535 caracteres.
- ***BINARY(size)***: *byte string* binario de longitud fija. Tamaño de 0 a 255 *bytes* (por defecto 1).
- ***VARBINARY(size)***: *byte string* binario de longitud variable. Tamaño (máximo) de 0 a 65.535 *bytes*.
- ***TINYTEXT***: *string* de máximo 255 caracteres.
- ***TINYBLOB***: *BLOB* de máximo 255 *bytes*.
- ***TEXT(size)***: *string*. Tamaño (máximo) puede ser hasta 65.535 caracteres.
- ***BLOB(size)***: *BLOB*. Tamaño (máximo) puede ser hasta 65.535 *bytes*.
- ***MEDIUMTEXT***: *string* de máximo 16.777.215 caracteres.
- ***MEDIUMBLOB***: *BLOB* de máximo 16.777.215 *bytes*.
- ***LONGTEXT***: *string* de máximo 4.294.967.295 caracteres.
- ***LONGBLOB***: *BLOB* de máximo 4.294.967.295 *bytes*.
- ***ENUM(val1, val2, val3, ...)***: *string* que solo puede tener un valor de entre los indicados (se pueden indicar hasta 65.535 valores distintos). Si se inserta un valor distinto a los indicados, el resultado insertado será un valor vacío. También es posible insertar un número que se refiere al orden de los valores (*1-based index*). En este caso el valor 2 corresponde a ***val2***.
- ***SET(val1, val2, val3, ...)***: *string* que puede contener 0 o más valores de entre los indicados, separados por comas. Los valores indicados no pueden contener comas. Se pueden indicar hasta 64 valores. En este caso, insertar un número funciona de forma distinta a ***ENUM***: cada valor es representado por un bit del índice, es decir, ***val1*** tiene el índice 1, ***val2*** el 2, ***val3*** el 4, etc. El índice 3 comprendería tanto ***val1*** como ***val2***. El ***SET*** no puede indicar valores duplicados. Aunque se inserte un valor conteniendo varios elementos iguales en la lista, solo se tendrá en cuenta uno de ellos.

Un *BLOB* es un objeto binario grande (*binary large object*). Los distintos campos ***TEXT*** y ***BLOB*** se pueden almacenar fuera de la tabla, de tal modo que esta almacena solo un apuntador. Además no pueden formar parte de un índice. Son más lentos que ***CHAR*** o ***VARCHAR***, ***BINARY*** o ***VARBINARY***, que se almacenan *inline* en la tabla y sí pueden formar parte de un índice.

### Numéricos

- ***BIT(size)***: un entero con el número de bits especificado (máximo 64, por defecto 1).
- ***TINYINT(size)***: un entero entre -128 y 127, o de 0 to 255 sin signo.
- ***BOOL*** o ***BOOLEAN***: booleano. 0 es falso, y cualquier otro número es verdadero.
- ***SMALLINT(size)***: un entero entre -32.768 y 32.767, o de 0 to 65.535 sin signo.
- ***MEDIUMINT(size)***: un entero entre -8.388.608 y 8.388.607, o de 0 to 16.777.215 sin signo.
- ***INT(size)*** o ***INTEGER(size)***: un entero entre -2.147.483.648 y 2.147.483.647, o de 0 to 4.294.967.295 sin signo.
- ***BIGINT(size)***: un entero entre -9.223.372.036.854.775.808 y 9.223.372.036.854.775.807, o de 0 to 18.446.744.073.709.551.615 sin signo.
- ***FLOAT***: punto flotante de simple precisión (4 *bytes*).
- ***DOUBLE***: punto flotante de doble precisión (8 *bytes*).
- ***DECIMAL(size, d)*** o ***DEC(size, d)***: números reales con precisión exacta. ***size*** indica el número total de dígitos significativos (máximo 65, por defecto 10), y ***d*** indica cuántos de ellos son decimales (máximo 30, por defecto 0).

El argumento de los enteros indica solamente el ancho máximo (columnas de texto) a usar al mostrar la columna (que será máximo 255).

Los datos numéricos tienen una opción extra: ***UNSIGNED*** (no negativos) o ***ZEROFILL*** (rellena con ceros a la izquierda), que se añaden justo después del tipo. ***ZEROFILL*** implica automáticamente ***UNSIGNED***.

### Fecha y hora

- ***DATE***: formato ***YYYY-MM-DD***. Rango soportado: de '1000-01-01' a '9999-12-31'.
- ***DATETIME***: formato ***YYYY-MM-DD hh:mm:ss***. Rango soportado: de '1000-01-01 00:00:00' a '9999-12-31 23:59:59'.
- ***TIMESTAMP***: número de segundos desde la época *Unix* ('1970-01-01 00:00:00' UTC). Número entero o mismo formato que ***DATETIME***. Rango soportado de '1970-01-01 00:00:01' a '2038-01-09 03:14:07'.
- ***TIME***: formato ***hh:mm:ss***. Rango soportado de '-838:59:59' a '838:59:59'.

A los tipos ***DATETIME*** y ***TIMESTAMP*** se les puede añadir `DEFAULT CURRENT_TIMESTAMP` y/o `ON UPDATE CURRENT_TIMESTAMP` en la definición de la columna para dar como valor por defecto la hora de creación del registro y/o para que se actualice automáticamente el valor respectivamente.

## Motores

En el caso de *MySQL*/*MariaDB*, suelen existir varios motores de base de datos, entre los que destacan:

- *MyISAM*: por defecto. Más rápido para consultas (`SELECT`), eficiente en cuanto a espacio en disco y memoria *RAM*, y mejor para búsquedas de textos largos. El problema es que no aplica mecanismos de integridad referencial.
- *InnoDB*: mejor para manipulación de datos (inserciones, ediciones, borrados). Aplica mecanismos de integridad referencial. Es el motor a usar para bases de datos relacionales.

El motor se define **a la hora de crear la tabla**. Más adelante ya no se puede cambiar. Por tanto, puede haber tablas dentro de la misma base de datos que usen distintos motores.

Para definir el motor por defecto al crear una tabla, se hace en el archivo de configuración (***my.ini*** o ***my.cnf*** dependiendo del sistema):

```
default-storage-engine=MyISAM
```

O bien:

```
default-storage-engine=InnoDB
```

También se puede especificar el motor en el momento de crear la tabla, usando `ENGINE` (véase más abajo).

Para saber qué motor está definido por defecto, en la consola del SGBD teclearemos:

```sql
SELECT @@default_storage_engine;
```

Para saber qué motor usa cada tabla:

```sql
SHOW TABLE STATUS;
```

Para saber qué motor usa una tabla concreta:

```sql
SHOW TABLE STATUS WHERE Name = 'usuarios';
```

## Estructura de la BD

```sql
CREATE DATABASE blog;
SHOW DATABASES;
USE blog;
SHOW TABLES;
DROP DATABASE blog;
```

```sql
CREATE TABLE usuarios(
    id int(11) AUTO_INCREMENT NOT NULL,
    nombre varchar(100) NOT NULL,
    apellidos varchar(255) NULL,
    edad int(3),
    email varchar(100) NOT NULL,
    password varchar(255) DEFAULT '1234',
    CONSTRAINT pk_usuarios PRIMARY KEY(id),
    CONSTRAINT uq_email UNIQUE(email)
) ENGINE=InnoDB;
DESCRIBE usuarios;
TRUNCATE TABLE usuarios;
DROP TABLE usuarios;
DROP TABLE IF EXISTS usuarios;
```

Por defecto, ***NULL***. Solo puede haber un campo ***AUTO_INCREMENT*** y debe ser **clave primaria**.

```sql
ALTER TABLE users RENAME TO usuarios;
ALTER TABLE usuarios
    CHANGE surnames apellidos varchar(159) NULL;
ALTER TABLE usuarios MODIFY apellidos varchar(255) NULL;
ALTER TABLE usuarios ADD telefono varchar(20) NOT NULL;
ALTER TABLE usuarios ADD CONSTRAINT uq_tel UNIQUE(telefono);
ALTER TABLE usuarios DROP telefono;
```

Tanto en la creación como en la modificación, las restricciones se pueden indicar sin `CONSTRAINT <nombre>`, pero entonces no nos podemos referir a esa restricción por nombre.

Tabla que tiene una *foreign key* que referencia a la tabla ***usuarios***:

```sql
CREATE TABLE entradas(
    # ...
    usuario_id int(11) NOT NULL,
    #...
    CONSTRAINT fk_entrada_usuario FOREIGN KEY(usuario_id)
        REFERENCES usuarios(id)
        ON DELETE CASCADE
        ON UPDATE RESTRICT
);
```

Las opciones para `ON DELETE` y `ON UPDATE`:

- ***RESTRICT*** (o ***NO ACTION***) rechaza la acción. Es la opción por defecto.
- ***SET NULL*** establece el valor apuntado a ***NULL***.
- ***CASCADE*** propaga el cambio (borrado o actualización).
- ***SET DEFAULT*** establece el valor apuntado a su valor por defecto. El motor *InnoDB* (por defecto en *MySQL*) no acepta esta opción.

## Manipulación de datos

*SQL* (*Structured Query Language*) tiene un subconjunto que forma un *DML* (*Data Manipulation Language*), el cual debe contener por definición sentencias para las operaciones ***CRUD***: inserción (`INSERT`, *Create*), selección (`SELECT`, *Read*), modificación (`UPDATE`, *Update*) y borrado (`DELETE`, *Delete*).

### Inserciones

```sql
INSERT INTO usuarios VALUES(
    null, 'Pepe', 'Martinez', 25, 'info@pepe.com', '1234');
INSERT INTO usuarios(nombre, email)
    VALUES('Pepe', 'info@pepe.com');
```

Se puede indicar ***null*** para ***id*** porque es autoincremental. El número de argumentos a `VALUES()` debe ser exacto, independientemente de si los campos tienen valor por defecto.

### Selección de datos

```sql
SELECT * FROM usuarios;
SELECT email, apellidos, nombre FROM usuarios;
SELECT nombre, 54+6 FROM usuarios;
SELECT nombre, edad+5 FROM usuarios;
SELECT nombre, edad+15 AS muchaedad FROM usuarios;
SELECT nombre, edad+15 AS muchaedad FROM usuarios
    ORDER BY muchaedad;
SELECT nombre, edad+15 AS muchaedad FROM usuarios
    ORDER BY muchaedad ASC;
SELECT nombre, edad+15 AS muchaedad FROM usuarios
    ORDER BY muchaedad DESC;
```

Las expresiones (incluyendo los simples nombres de campo) se pueden agrupar en paréntesis. Las expresiones pueden incluir funciones *SQL*. Se admiten los operadores aritméticos habituales (`+`, `-`, `*`, `/`, `%`).

Las cláusulas `WHERE` no tienen acceso a los alias (`AS`), pero `HAVING` sí. El orden por defecto es ***ASC***.

Si en lugar de `SELECT` indicamos `SELECT DISTINCT`, la consulta no retornará registros duplicados.

### Funciones *SQL*

Ejemplos de algunas funciones *SQL*:

*Aggregate functions*: `AVG()`, `COUNT()`, `FIRST()`, `LAST()`, `MAX()`, `MIN()`, `SUM()`.

Funciones matemáticas: `ABS()`, `CEIL()`, `FLOOR()`, `TRUNCATE()`, `ROUND()`, `MOD()`, `POWER()`, `SQRT()`, `RAND()`.

Funciones para textos: `UCASE()` o `UPPER()`, `LCASE()` o `LOWER()`, `MID()` (extrae *substrings*), `LENGTH()`, `CONCAT()`, `TRIM()`, `STRCMP()`.

Funciones para fechas: `NOW()` (fecha + hora), `CURDATE()` (fecha), `CURTIME()` (hora), `DATEDIFF()`, `DAYNAME()`, `DAYOFMONTH()`, `DAYOFWEEK()`, `DAYOFYEAR()`, `DAY()`, `MONTH()`, `YEAR()`, `HOUR()`, `MINUTE()`, `SECOND()`, `DATE_FORMAT()`.

Otras funciones: `FORMAT()`, `ISNULL()`, `IFNULL()`, `VERSION()`, `USER()`, `DATABASE()`

Las funciones que retornan un booleano, retornan en realidad 0 o 1.

### Condiciones

```sql
SELECT * FROM usuarios WHERE email = 'pepe@pepe.com';
```

Operadores de comparación: `=`, `!=`, `>`, `<`, `>=`, `<=`, `IN`, `BETWEEN ... AND` (inclusive), `IS NULL`, `LIKE`. Cualquiera de estas expresiones lógicas se puede negar añadiendo `NOT` delante (excepto en el caso de `expr IS NULL`, que pueda negarse como `NOT expr IS NULL` o como ` expr IS NOT NULL`). Para encadenar expresiones se puede usar `AND` y `OR`, y agruparlas entre paréntesis cuando sea necesario.

`expresion IS NULL` equivale a `ISNULL(expresion)`.

```sql
SELECT * FROM usuarios WHERE YEAR(fecha)
    NOT IN (2019, 2020);
SELECT * FROM usuarios WHERE YEAR(fecha)
    NOT IN (2019, 2020) OR id != 4;
SELECT * FROM usuarios WHERE YEAR(fecha)
    NOT BETWEEN 2019 AND 2020 AND id != 4;
```

En el último caso, el primer `AND` forma parte del `BETWEEN`. El segundo es el operador lógico. Los conjuntos (*sets*) deben ir entre paréntesis. Puede ser una subconsulta (un `SELECT`, de hecho, genera un conjunto).

Los comodines *wildcards* de `LIKE` son:

- `%` representa cualquier cantidad de caracteres (0 o más).
- `_` representa un carácter (exactamente uno).

Si **al final de todo** indicamos `LIMIT <N>`, la consulta retornada se limitará a los primeros ***N*** registros. Si se indica `LIMIT <N>,<M>` la consulta mostrará desde el elemento ***N*** (*zero-based*), y un total de ***M*** elementos.

### Actualización y eliminación de registros

```sql
UPDATE usuarios SET nombre='Pepe';
UPDATE usuarios SET nombre='Pepe' WHERE id=3;
UPDATE usuarios SET nombre='Pepe',
    email='info@pepe.com' WHERE id=3;
```

La primera sentencia actualiza **todos** los registros.

```sql
DELETE FROM usuarios;
DELETE FROM usuarios WHERE id=3;
```

La primera sentencia elimina **todos** los registros.

### Agrupamiento

Se agrupan en uno todos los registros tales que sus campos definidos en la cláusula `GROUP BY` sean todos coincidentes. Supongamos que tenemos esta tabla llamada ***coches***:

| marca | modelo | color    | km     |
| :---- | :----- | :------- | :----- |
| SEAT  | Ibiza  | blanco   | 150000 |
| SEAT  | Ibiza  | amarillo | 120000 |
| SEAT  | Ibiza  | negro    | 110000 |
| SEAT  | Panda  | rojo     | 170000 |
| VW    | Polo   | blanco   | 100000 |
| VW    | Polo   | negro    | 80000  |
| VW    | Passat | azul     | 120000 |

Veamos el resultado de algunas consultas:

```sql
SELECT marca FROM coches;
```

| marca |
| :---- |
| SEAT  |
| SEAT  |
| SEAT  |
| SEAT  |
| VW    |
| VW    |
| VW    |

```sql
SELECT marca FROM coches GROUP BY marca;
```

| marca |
| :---- |
| SEAT  |
| VW    |


```sql
SELECT marca, modelo FROM coches;
```

| marca | modelo |
| :---- | :----- |
| SEAT  | Ibiza  |
| SEAT  | Ibiza  |
| SEAT  | Ibiza  |
| SEAT  | Panda  |
| VW    | Polo   |
| VW    | Polo   |
| VW    | Passat |

```sql
SELECT marca, modelo FROM coches GROUP BY marca, modelo;
```

| marca | modelo |
| :---- | :----- |
| SEAT  | Ibiza  |
| SEAT  | Panda  |
| VW    | Polo   |
| VW    | Passat |

En principio, los campos del `SELECT` deberían ser los mismos que los del `GROUP BY`. En el caso concreto de *MySQL*/*MariaDB*, si el modo ***ONLY_FULL_GROUP_BY*** está activado y la consulta tiene una cláusula `GROUP BY`, solo se puede hacer referencia en la lista `SELECT` y en la condición `HAVING` a columnas agrupadas, es decir, o bien campos incluidos en la lista `GROUP BY`, o bien *aggregate functions* sobre cualquier campo (o bien ambas cosas).

Para establecer el modo mencionado, se incluirá en el archivo de configuración de la BD:

```
sql-mode="ONLY_FULL_GROUP_BY"
```

Si se quieren activar varios modos, se separarán por comas.

Ejemplo seleccionando campos en `SELECT` que no aparecen en `GROUP BY` (no siempre aceptado por el SGBD):

```sql
SELECT marca, modelo, color FROM coches
    GROUP BY marca, modelo;
```

| marca | modelo | color    |
| :---- | :----- | :------- |
| SEAT  | Ibiza  | blanco   |
| SEAT  | Panda  | rojo     |
| VW    | Polo   | blanco   |
| VW    | Passat | azul     |

Por ejemplo, en el primer registro resultante, el color seleccionado es blanco, porque es el primer valor del grupo, pero también hay un SEAT Ibiza amarillo y uno negro. Este tipo de consultas no resulta, pues, recomendable.

En el caso inverso, en el que hay campos en `GROUP BY` que no se seleccionan en `SELECT`:

```sql
SELECT marca FROM coches GROUP BY marca, modelo;
```

| marca |
| :---- |
| SEAT  |
| SEAT  |
| VW    |
| VW    |

En este caso, la marca aparece una vez por cada modelo distinto de la misma, lo cual tiene poco sentido.

En todo caso, siempre se pueden seleccionar funciones *aggregate* sobre campos no agrupados:

```sql
SELECT marca, modelo, MIN(km) FROM coches
    GROUP BY marca, modelo;
```

| marca | modelo | km     |
| :---- | :----- | :----- |
| SEAT  | Ibiza  | 110000 |
| SEAT  | Panda  | 170000 |
| VW    | Polo   | 80000  |
| VW    | Passat | 120000 |

Puede ser útil utilizar un alias para columnas agregadas con *aggregate functions*. Si queremos incluir una cláusula `WHERE`, esta debe ir siempre **antes** del `GROUP BY`. Para actuar sobre columnas agregadas, se usará `HAVING`. `WHERE` no acepta alias ni columnas agrupadas. Si hay `GROUP BY`, la cláusula `HAVING` no puede ir antes de esta.

```sql
SELECT marca, modelo, MIN(km) as kmmin FROM coches
    GROUP BY marca, modelo
    HAVING kmmin > 100000;
```

En el caso específico de `COUNT()`, como no nos interesa el valor de ningún campo sino solamente el número de registros de cada grupo, podemos incluir como argumento el nombre de cualquier campo, o simplemente un asterisco: `COUNT(*)`.

### Uniones

```sql
SELECT nombre, apellidos FROM usuarios
UNION
SELECT marca, modelo FROM coches;
```

El nombre de las columnas resultantes es el del primer `SELECT`. Los dos (o más, si los hay) `SELECT` implicados deben tener el mismo número de columnas.

### Subconsultas

Supongamos que cada registro en la tabla ***coches*** tiene un campo ***usuario_id*** que es una clave externa a la tabla ***usuarios***.

Todos los coches de Pepe o Paco:

```sql
SELECT marca, modelo FROM coches
    WHERE usuario_id IN
    (SELECT id FROM usuarios
        WHERE nombre = 'Pepe'
        OR nombre = 'Paco');
```

Usuarios con más de un coche:

```sql
SELECT nombre FROM usuarios WHERE id IN
    (SELECT usuario_id FROM coches GROUP BY usuario_id
        HAVING COUNT(*) > 1);
```

```sql
SELECT * FROM tabla WHERE EXISTS (SELECT ...);
```

Si la subconsulta retorna un solo valor, se puede considerar como tal, y comparar usando los operadores de igualdad, etc.

### Joins

```sql
SELECT * FROM usuarios, coches;
```

Este tipo de consulta es un producto cruzado (*cross product* o *cross join*), y equivale a:

```sql
SELECT * FROM usuarios CROSS JOIN coches;
```

Supongamos que tanto la tabla ***usuarios*** como la ***coches*** tienen un campo ***nombre***. Para referirnos a tal campo hay que desambiguar:

```sql
SELECT usuarios.nombre, coches.nombre, modelo
    FROM usuarios, coches;
```

En el caso del campo ***modelo*** no hace falta, aunque se podría indicar también ***coches.modelo***.

Para hacer alias de tablas:

```sql
SELECT us.nombre, co.nombre, modelo
    FROM usuarios us, coches co;
```

Si tenemos 3 tablas ***t1***, ***t2*** y ***t3*** con 50 registros cada una, entonces `t1, t2, t3` tendrá 125000 registros (todas las combinaciones de registros). Aunque hagamos un `WHERE` posterior, la tabla inicial es monstruosa.

Para evitar que se generen estas tablas tan grandes, están los *joins*:

```sql
SELECT apellidos, marca, modelo FROM
    usuarios us INNER JOIN coches co
    ON us.id = co.usuario_id;
```

Solo aparecen en la tabla los registros que asocian un coche a un usuario. Los coches que no tienen usuario (***usuario_id*** es ***null***) no aparecen, y tampoco los usuarios que no están asociados a un coche.

Si hacemos `RIGHT JOIN` en lugar de `INNER JOIN`, entonces se incluirán, además, todos los coches que no están asociados a un usuario, con los campos correspondientes al usuario a ***null***.

Del mismo modo, con `LEFT JOIN` aparecerán, además de los registros del `INNER JOIN`, todos los usuarios sin coche, con los campos relativos al coche a ***null***.

En el caso de `FULL OUTER JOIN` aparecerán tanto los registros del `LEFT JOIN` como los del `RIGHT JOIN`.

Modos de escribir los *joins*:

- `CROSS JOIN`
- `(INNER) JOIN`
- `LEFT (OUTER) JOIN`
- `RIGHT (OUTER) JOIN`
- `FULL (OUTER) JOIN`

El único que no tiene cláusula `ON` es el `CROSS JOIN`.

### Vistas

Siempre actualizadas, ya que usan los datos originales.

```sql
CREATE VIEW usu_coches AS SELECT ...;

SELECT * FROM usu_coches;
SELECT * FROM usu_coches WHERE ...;

DROP VIEW usu_coches;
```

Una vista aparece como una tabla más de la BD.

## PHP + MySQL/MariaDB

Hay básicamente dos extensiones *PHP* disponibles: *mysqli* (*MySQL improved*) y *PDO_MySQL*. Nos centraremos en la primera.

### Crear la conexión

Es posible utilizar una interfaz procedural:

```php
$mysql = mysqli_connect("ejemplo.com", "usuario",
                        "password", "database", 3306);
$result = mysqli_query($mysql, "SELECT modelo FROM coches");
$registro = mysqli_fetch_assoc($result);
```

O una interfaz orientada a objetos:

```php
$mysql = new mysqli("ejemplo.com", "usuario",
                    "password", "database", 3306);
$result = $mysql->query("SELECT modelo FROM coches");
$registro = $result->fetch_assoc();
```

No es recomendable mezclar ambos estilos (por claridad).

Al crear la conexión (función `mysqli_connect()` o constructor de ***mysqli***), el primer parámetro (servidor) puede ser un nombre de *host* o una derección *IP*. El quinto parámetro no es necesario indicarlo normalmente, a no ser que el servidor esté en un puerto distinto a 3306.

De hecho todos los argumentos en la creación de la conexión pueden omitirse, en cuyo caso se usarán los valores por defecto, que deben estar definidos en el archivo de configuración de *PHP*.

Si hay error, la función o método (constructor) de creación de la conexión retorna ***false***.

Para obtener el posible error al crear la conexión disponemos de las propiedades ***connect_error*** y ***connect_errno***, que contienen el mensaje de error y el número de error producido respectivamente. En el contexto procedural tenemos en su lugar las funciones `mysqli_connect_error()` y `mysqli_connect_errno()`, sin parámetros. Si la operación de creación de la conexión tiene éxito, el mensaje de error es ***NULL*** y el número de error es 0.

Una vez la conexión ha tenido éxito, las subsiguientes operaciones afectan a las propiedades (del objeto conexión) ***error*** (mensaje de error) y ***errno*** (número de error). En el modo procedural se consigue el mismo resultado con las funciones `mysqli_error()` y `mysqli_errno()` respectivamente, a las que se debe pasar como argumento el objeto conexión.

### Consultas

Al hacer una consulta, el cliente (*PHP*) puede obtener los resultados en un *buffer* (liberando recursos del servidor *MySQL*/*MariaDB*), o tras la consulta ir leyendo los registros uno a uno, lo cual mantiene más ocupado al servidor (no recomendado).

El método `query()` del objeto conexión o la función `mysqli_query()` realizan una consulta *buffered*, es decir obtienen los resultados en un *buffer* (siempre que se trate de una consulta `SELECT`, claro). El método recibe un argumento *string* que contiene la consulta. La función necesita dos argumentos: el primero es el objeto conexión, y el segundo la consulta.

El objeto consulta retornado (de tipo ***mysqli_result***) contiene los registros de la consulta, aunque no se puede acceder a estos mediante sintaxis de *array*. Sí se pueden obtener mediante `foreach`. Para acceder a un registro concreto se usa el método (del objeto consulta) `data_seek()`, que posiciona el **apuntador de registro actual** dentro de ese objeto (al crearse la consulta queda en 0, es decir, el primer registro). Este método recibe simplemente un entero. Si se quiere usar el modo procedural, disponemos de la función `mysqli_data_seek()` que recibe como parámetros el objeto consulta y el entero índice.

Si la creación de la *query* falla, el método o la función retornan ***false***.

El método `fetch_assoc()` (sin argumentos) del objeto consulta retorna un *array* asociativo (*string* => *string*) con el contenido del registro actual, y mueve el apuntador de registro actual una posición. En el modo procedural se usa la función `mysqli_fetch_assoc()`, que recibe como argumento el objeto consulta.

En cualquiera de los dos modos, tras un *fetch assoc* del último elemento de la consulta, los subsiguientes *fetch assoc* retornarán ***NULL***.

Si en lugar de obtener un *array* asociativo deseamos obtener un **objeto**, con los nombres de sus propiedades iguales a los nombres de los campos, usaremos el método `fetch_object()`, o la función `mysqli_fetch_object()`. En este caso hay que pasarle como argumentos:

- En el caso de la función, objeto resultado (***mysqli_result***). En caso del método, es precisamente ese objeto ***mysqli_result*** el que posee dicho método.
- El nombre de la clase del objeto creado. Por defecto, ***stdClass***.
- Un *array* opcional de parámetros a pasar al constructor del objeto instanciado.

El objeto retornado será ***NULL*** si no hay más filas, o ***false*** en caso de error.

Por otro lado, para saber cuántos registros contiene el resultado, existe la propiedad ***num_rows*** del objeto consulta, o la función `mysqli_num_rows()` a la que se debe pasar el objeto consulta.

Si las consultas pueden contener caracteres no *ASCII* (caracteres acentuados, ***ñ***, etc.) se debe mandar al servidor la consulta ***SET NAMES 'utf8'***.

Ejemplo OO:

```php
$mysql = new mysqli("ejemplo.com", "usuario",
                    "password", "database");
$mysql->query("SET NAMES 'utf8'");
$result = $mysql->query("SELECT modelo FROM coches");
// Número de registros:
echo $result->num_rows;
echo '<br/>';
// Todos los registros:
foreach($result as $row)
    echo $row['id'] . '<br/>';  // campo id del registro
// Un registro concreto:
$resul->data_seek(3);
$row = $result->fetch_assoc();
echo $row['id'];
```

Ejemplo procedural:

```php
$mysql = mysqli_connect("ejemplo.com", "usuario",
                        "password", "database");
mysqli_query($mysql, "SET NAMES 'utf8'");
$result = mysqli_query($mysql, "SELECT modelo FROM coches");
// Número de registros:
echo mysqli_num_rows($result);
echo '<br/>';
// Todos los registros:
foreach($result as $row)
    echo $row['id'] . '<br/>';  // campo id del registro
// Un registro concreto:
mysqli_data_seek($result, 3);
$row = mysqli_fetch_assoc($result);
echo $row['id'];
```

## PHP + Microsoft SQL Server

Para conectar a bases de datos *SQL Server* tenemos la complejidad adicional de que dependiendo del sistema operativo deberemos utilizar una u otra extensión *PHP*. Para entornos *Windows*, véase la documentación de la extensión *SQLSRV*, y para *Linux*, la extensión *PDO*.
