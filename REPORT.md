# Reporte Técnico: Análisis del Exploit "Copy Fail"

### ¿Cuál es el bug raíz y en qué archivo/función está?
El problema reside en el archivo crypto/algif_aead.c, en la función _aead_recvmsg(). En resumen, el kernel escribía y leía la misma porción de memoria a la vez durante una operación criptográfica, lo que permitía cambiar datos de archivos importantes cargados en memoria, por ejemplo `/usr/bin/su.

### ¿Por qué el write a dst es peligroso?
La razón es que /usr/bin/su es un programa especial cuyo funcionamiento permite ejecutarlo en modo setuid, es decir, posee privilegios de superusuario. Si conseguimos cambiar solo unos cuantos bytes de ese binario cuando está cargado en memoria, podemos lograr que se ejecute incorrectamente y así obtener acceso root.

Lo que hace el exploit es exactamente eso: cambia unos pocos bytes en el momento adecuado para modificar el comportamiento del programa y abrir una shell con privilegios de root.

### ¿Por qué el exploit es “stealthy”?
Porque no cambia nada en el disco. El archivo original no se modifica y su hash permanece igual. Solo se altera la copia temporal que el kernel mantiene en la RAM, conocida como page cache.

Al no existir cambios permanentes en el sistema de archivos, todo parece normal. Incluso después de reiniciar el equipo, las modificaciones desaparecen automáticamente.

### Conexión con lo que vimos en clase
El page cache es una copia de los archivos que mantiene el kernel en RAM para no estar leyendo continuamente desde el disco. Esta vulnerabilidad aprovecha precisamente ese mecanismo, ya que permite modificar la copia en memoria sin pasar por los controles habituales del sistema de archivos.

El bit setuid que configuramos desde chmod es fundamental para que funcione el ataque. Si /usr/bin/su no tuviese ese permiso especial, modificar el binario in memoria no permitiría obtener privilegios de administrador. El permiso -rwsr-xr-x es lo que convierte este bug en una auténtica escalada de privilegios.

Los inodos almacenan información importante del archivo: permisos, propietario y metadatos. Pero el ataque solo afecta a la copia en memoria, nunca al archivo real en disco, por lo que los inodos permanecen intactos y el sistema mantiene una apariencia normal.

### ¿Qué aprendiste?
Este ejercicio me sirvió para entender cómo muchas vulnerabilidades críticas pueden surgir a partir de pequeños errores que parecen inofensivos. En concreto, el cambio introducido en 2017 parecía una simple optimización y pasó desapercibido durante años.

También me ayudó a relacionar mejor conceptos vistos en clase, como el funcionamiento del page cache, el bit setuid y los inodos. Ver cómo todos estos mecanismos interactúan entre sí permite comprender mejor por qué una vulnerabilidad como esta puede convertirse en un problema de seguridad tan serio.
