# Sintaxis básica

*PHP* es un lenguaje para el desarrollo *web*. Los archivos *PHP* tienen extensión ***.php***. El código *PHP* se incluye entre etiquetas `<?php` ... `?>`. El archivo puede consistir en una sola de estas etiquetas, incorporando dentro todo un *script*, o también puede consistir en código *markup* (como *HTML*) que incluya etiquetas *PHP*.

En todo caso, el código *PHP* se ejecuta por entero en el servidor *web*, de tal modo que cuando llega el documento al navegador cliente, no hay rastro de él.

La etiqueta `<?=` equivale a `<?php echo`.

En algunos servidores puede estar habilitada la etiqueta corta `<? ... ?>`.

Si el archivo termina en etiqueta de cierre de bloque *PHP*, es recomendable no incluir la etiqueta de cierre final para evitar espacio blanco o *newlines* no deseados.

Cada instrucción debe finalizar con un `;`, exceptuando la última instrucción en un bloque (etiqueta) *php*, en cuyo caso no es necesario (la *closing tag* implica un punto y coma).

La sentencia `echo` produce salida, y no produce *newline* al final. Es una construcción del lenguaje y no una función, con lo que no son necesarios los paréntesis alrededor de los argumentos. Estos pueden ser de cualquier tipo y tantos como se desee. Por otro lado, la sentencia `print` es como `echo`, pero solo admite un argumento, que debe ser de tipo *string*.

Los *newlines* fuera de los bloques *php* son enviados a la salida.

En cuanto a los **comentarios**, se admiten los comentarios de una sola línea con `// ...`, o multilínea (no anidable) con `/* ... */`. A parte de estos comentarios tipo *C*/*C++*, también admite el estilo *shell script*, es decir ***#*** (más usado en archivos *PHP* que definen configuraciones).

A parte de ejecutarse en un servidor *web*, también es posible ejecutar *scripts PHP* localmente (en línea de comandos), utilizando el comando `php`.
