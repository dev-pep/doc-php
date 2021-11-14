# Extensión ldap

La directiva ***ldap.max_links*** en ***php.ini*** indica el número máximo de conexiones *ldap* por proceso. Por defecto es -1 (ilimitadas).

La secuencia típica de acciones es primero crear una conexión (`ldap_connect()`), luego hacer un *bind* a través de un usuario (`ldap_bind()`), y tras realizar las operaciones deseadas, cerrar la conexión (`ldap_close()`).

Algunos servidores permiten un *bind* anónimo de solo lectura.

En todo caso, para acceder al directorio *ldap* será necesario (a parte del posible usuario para *bind*) disponer de la dirección del servidor y del nombre base (*base name*).

El nombre base es el que se tomará como raíz de las búsquedas. Indicarlo puede ser útil cuando el servidor solo incluye una parte del mundo (una rama concreta, de una provincia, por ejemplo) y no están permitidos los *referrals* (acceso a otros servidores cuando el directorio está compartimentado en dominios distintos).
