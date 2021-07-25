# Sintaxis básica

Código *PHP* entre etiquetas `<?php ... ?>`. Etiqueta `<?=` equivale a `<?php echo`.

En algunos servidores puede estar habilitada la etiqueta corta `<? ... ?>`.

Si el archivo termina en etiqueta de cierre de bloque *PHP*, es recomendable no incluir la etiqueta de cierre final para evitar espacio blanco o *newlines* no deseados.

Cada instrucción debe finalizar con un `;`, exceptuando la última instrucción en un bloque *php*, en cuyo caso no es necesario (la *closing tag* implica un punto y coma).

La función `echo` no produce *newline* al final. Los *newlines* fuera de los bloques *php* son enviados a la salida.

En cuanto a los **comentarios**, se admiten los comentarios de una sola línea con `// ...`, o multilínea (no anidable) con `/* ... */`.
