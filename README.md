# Computacion_Aplicada-Grupo_3_-Los_Tres_Bits-

## Integrantes: 
## Procedimientos llevados a cabo para la configuración de la máquina virtual
### Preparación: Importar la máquina virtual base
El desarrollo del trabajo parte de una máquina virtual previamente configurada, con el sistema operativo Debian 11. La misma puede importarse al gestor de máquinas virtuales a través de los archivos ‘.OVA’ correspondientes. Éstos proveen la imagen de la máquina y sus especificaciones tanto a nivel de dispositivo físico como dentro de la propia máquina virtual. Una vez importada, por fuera de la máquina virtual deben hacerse cambios mínimos que serán explicados a su debido momento durante el desarrollo.

### 1) Configuración del entorno
#### Clave Root:
Para poder manipular la máquina virtual, se debe primero obtener la contraseña del usuario Root para llevar a cabo las diversas configuraciones a nivel de sistema. Para ello, se deben modificar las instrucciones de arranque para que inicie el intérprete de comandos como primer paso. 

Al iniciar la máquina virtual, el gestor de arranque GRUB presenta las opciones disponibles. Entre las funciones del usuario, presionando la tecla ‘e’, puede modificar las órdenes de arranque. Entre dichos parámetros, el comando ‘linux’ es el disparador del arranque del sistema operativo. Si se agrega al final de su línea ‘init=/bin/bash’, al iniciar se dispone con una interfaz para ejecutar comandos como usuario Root. 
Para poder realizar cambios persistentes, mediante el comando ‘mount -o remount, rw /’ se remontan las unidades en modo lectura y escritura. La clave de Root puede cambiarse luego con el comando ‘passwd root’, y puede reiniciarse ejecutando la instrucción ‘exec /sbin/init’.

#### Hostname: 
Para actualizar el hostname de forma persistente, debe actualizarse en el archivo /etc/hostname, cuyo contenido debe ser el nombre de host que quiera usarse.

#### Extra: Actualizar repositorios para la descarga de paquetes
Debian 11 llegó al fin de su ciclo de vida, por lo que los paquetes fueron movidos a los ‘Debian Archives’, debiendo usar las siguientes URLs el gestor de paquetes. Los contenidos deben actualizarse en el archivo /etc/apt/sources.list:
deb http://archive.debian.org/debian bullseye main contrib non-free
deb http://archive.debian.org/debian-security/ bullseye-security main contrib non-free
deb http://archive.debian.org/debian bullseye-updates main contrib non-free

### 2) Servicios
SSH: Para utilizar SSH debe instalarse el paquete ‘openssh-server’. Los pasos para su configuración son los siguientes:
a)	Actualizar el hostname en /etc/hosts, si el que allí se encuentra no coincide con el nombre de host elegido.
b)	Copiar la clave pública desde el usuario cliente al host. Para este pasaje inicial de archivos una alternativa implica permitir la conexión estándar con el usuario root usando su contraseña, con el comando:
scp [ubicación clave publica] root@direcciónIP:/etc/.ssh/authorized_keys
c)	Otorgar los permisos adecuados para el directorio .ssh del usuario y al archivo .ssh/authorized_keys:
chmod 700 ~/.ssh (lectura, escritura, ejecución para el usuario)
chmod 600 ~/.ssh/authorized_keys (lectura y escritura para el usuario)
d)	Configurar SSH para que utilice únicamente el par de claves: El archivo /etc/ssh/sshd_config contiene las configuraciones a usar en el dispositivo en caso de ser el servidor. Allí, deben modificarse los parámetros:
permit root login: prohibit password (permite entrar como root, pero no con la contraseña)
password authentication: no (impedir de forma general el uso de contraseñas)
Una vez configurado, un cliente puede conectarse al host usando su clave privada, con el comando ssh -i [ubicación clave privada] usuario@ip. A su vez, los archivos faltantes pueden copiarse indicando la clave a usar de la misma manera.
Apache: Su uso requiere el paquete ‘apache2’. Una vez instalado, automáticamente entra en ejecución en el arranque, pudiendo verificar el funcionamiento buscando desde un navegador la dirección IP de la máquina virtual, lo que carga el archivo configurado a leer. Los archivos que lee por defecto encuentran dentro de /var/www/html, por lo que se deben copiar allí tanto el index.php como el logo.png, ya que este último es buscado por el index.php dentro de su mismo directorio.
Para que pueda utilizarse archivos .php, debe además instalarse el paquete ‘libapache2-mod-php’
MariaDB: Emplea los paquetes mariadb-server, mariadb-client y galera-4 para su gestión y funcionamiento, y php-mysqli y php-pdo para que pueda interactuar el archivo .php con la base de datos. 
Una vez descargado, con el comando ‘mariadb-secure-installation’ se realizan configuraciones iniciales seguras mediante una serie de prompts con las indicaciones recomendadas, como eliminar usuarios anónimos o tablas de prueba. 
Para cargar la base de datos provista, se utiliza el comando ‘mariadb -u root -p < [ubicación del .sql]’ para ejecutar con el usuario root de mariadb los contenidos del .sql especificado.
Para la conexión con la base de datos, el index.php utiliza un usuario y clave predefinidos, por lo que debe crearse en el gestor de base de datos:
-Se ingresa al programa de gestión con el usuario root: mariadb -u root -p
-Se crea el usuario: create user ‘lcars’@’localhost’ identified by ‘NCC1701D’

3) Configuración de red:
Por defecto, el dispositivo usa una dirección IP asignada mediante DHCP. Para configurar una dirección estática, se debe modificar el archivo /etc/network/interfaces: La línea que comienza con ‘iface’ define las configuraciones del dispositivo, como el nombre de interfaz, y el modo de direccionamiento. En lugar de, como tiene el ejemplo, dhcp, debe en su lugar escribirse ‘static’ para dirección estática, y en las siguientes líneas definir:
-	address   [dirección IP elegida]/[mascara de subred]
la dirección IP debe ser una disponible dentro del rango de direcciones de la red física, para que puedan reconocerse dentro de la red la máquina virtual y la máquina anfitriona, y la máscara de subred adecuada
-	gateway [default Gateway, el mismo que la máquina anfitriona]
-	dns-nameservers [misma dirección que el Gateway]
Una vez modificado el archivo, se deben aplicar los cambios con el comando ‘systectl restart networking.services’

4) Almacenamiento
Creación del disco virtual:
Para agregar el disco a la máquina virtual, debe primero crearse en el mismo VirtualBox: Dentro de las configuraciones de la máquina virtual, en el menú de almacenamiento, se debe agregar un disco duro al controlador SATA. En el panel de selector de discos duros, de elige la opción de crear, siguiendo los pasos de la ventana emergente. El tipo de archivo a usar es de VDI (VirtualBox Disk Image), y definir el tamaño que se quiere que tenga. El nuevo disco creado puede ser seleccionado en el selector de discos duros con un doble click, para añadirlo a la lista de dispositivos de la máquina virtual.
Configuración del dispositivo y sus particiones:
Una vez hecho, la máquina virtual reconoce la nueva unidad, pero esta debe configurarse para poder usarla: 
Con el comando ‘lsblk’ se pueden visualizar los dispositivos de almacenamiento disponibles y sus particiones, para poder así registrar el nombre del dispositivo agregado. Conociendo dicho nombre, sea por ejemplo ‘sdb’ o ‘sdc’, con el comando:
fdisk /dev/[nombreDelDisco] se inicializa un programa para la creación de particiones. Dicho comando inicia un menú interactivo, en donde se configura el disco mediante una serie de comandos. Para empezar, se indica que se busca crear nuevas particiones con la opción ‘n’, y seguido por la opción ‘p’ para indicar que son particiones primarias. Cuando el programa solicite el último sector de la partición, allí puede especificarse el tamaño con instrucciones de formato: ‘+nX, con ‘X’ representando la unidad para definir el tamaño (sea M para megabytes, o G para gigabytes, por ejemplo), y siendo ‘n’ la cantidad de dicha unidad que se quiere que tenga. La opción w es usada para guardar los cambios. 
Una vez creadas, al usar el comando lsblk las particiones del disco son visibles, cada una con un identificador propio.
Luego de crear las particiones, se debe asignar su file system, definir la estructura a usar para que puedan utilizarse adecuadamente. Para el caso, se usa ext4, definido con el comando:
mkfs.ext4 /dev/[partición a dar formato]
Por último, para que sean utilizables, deben estar montadas las particiones. Pueden montarse manualmente con el comando mount [dispositivo] [lugar de montaje], siendo el lugar de montaje un directorio dentro de /mnt creado previamente, pudiendo elegir el nombre a usar, como son /www_dir y /backup_dir. 
El montaje manual perdura durante la sesión únicamente. Para que la unidad se monte en sesiones futuras, debe agregarse al archivo /etc/fstab, en donde debe agregarse al final del mismo una línea para cada partición nueva que se quiera montar, con un formato:
UUID=[UUID dispositivo, visible ejecutando ‘blkid’] [punto de montaje] [file system usado] [opciones]
Ejemplo: UUID=UUID_del_dispositivo /mnt/www_dir  ext4 defaults (para usar opciones por defecto)
Migración de archivos de apache:
Una vez configuradas las nuevas particiones, pueden moverse los archivos de apache dentro de /mnt/www_dir. Para que apache lea los archivos desde esta nueva ubicación, deben modificarse dos archivos de configuración: 
-	/etc/apache2/apache2.conf: Contiene configuraciones globales de apache. Entre estas, otorga permisos de seguridad y capacidades a ciertos directorios:

<directory [directorio]>
             configuraciones sobre la ruta
</directory>

uno de estos corresponde al directorio donde se ubican los archivos para cargar en el sitio. Por defecto, está configurado para /var/www. Son permisos necesarios para el funcionamiento, por lo que debe cambiarse por /mnt/www_dir
-	/etc/apache2/sites-available/000-default.conf: Contiene configuraciones por defecto respecto al sitio, como el puerto en el que escucha, o el punto de lectura de archivos. La línea ‘DocumentRoot’ define el directorio raíz en donde buscar archivos para usar en el sitio. Aquí debe cambiarse nuevamente el contenido a /mnt/www_dir
Redirigir contenidos de /proc/partitions: Sus contenidos pueden copiarse sobre un nuevo archivo persistente, redirigido a /proc/partición con el comando:
cat proc/partitions > proc/partición (‘cat’ retorna los contenidos del archivo, y con el símbolo > se redirigen a otro archivo, existente o no)
Backup:
El script desarrollado se encuentra en /opt/scripts/backup_full.sh, teniendo que crear el archivo y su directorio. Como parámetros recibe el directorio que se quiera respaldar y, opcionalmente, el directorio destino dentro de /backup_dir, el cual debe existir a la hora de la ejecución del script. 
Previo a la compresión, se realizan validaciones previas sobre los parámetros y dispositivos: Que se cuente con los parámetros necesarios, que se encuentren las rutas de origen y destino (si la hay), y que las unidades asociadas estén disponibles para el uso
Para la creación del comprimido, se debe generar el nombre de archivo: 
-	A partir de la ruta origen recibida, se extrae con el comando ‘basename [ruta]’ el nombre del directorio seleccionado, sin su ruta asociada.
-	Con el comando ‘date +%Y%m%d’ se obtiene la fecha actual con el formato ‘YYYYMMDD’
-	La ruta destino se arma partiendo del directorio base /mnt/backup_dir, y si se recibe un segundo parámetro para el destino, se le concatena para que tenga el formato /mnt/backup_dir/[rutaDestino]
El comprimido resultante entonces se ubicaría en:
/mnt/backup_dir/[rutaOpcional]/[nombreDirectorio]_bkp_YYYYMMDD.tar.gz
Y la compresión se realiza con el comando ‘tar’ con las opciones ‘c’ (crear archivo nuevo), ‘z’ (comprimir usando gzip) y ‘f’ (se indica el nombre del archivo de salida). 
Eso sí, al comprimir un archivo, este mantiene su estructura de directorios, por lo que al descomprimirlo se genera una serie de directorios en el lugar de extracción igual a los directorios padre del comprimido, desde la raíz. Por ejemplo, comprimir el directorio ‘/root/textos/’, y extraerlo en ‘/root/test/testUno/’, resulta en una estructura de forma ‘/root/tests/testUno/root/textos/[contenidos]’
Para evitar que suceda, se usa la opción ‘-C [rutaOrígen] [contenidosAComprimir]’, para que la estructura de directorios parta de la ruta origen especificada, en lugar de la raíz. 
El comando utilizado entonces es el siguiente:
tar -czf “[ubicación donde comprimir]” -C “[directorio a comprimir” “.” 
Ciertos directorios tienen archivos que cambian constantemente, pudiendo cambiar incluso durante la misma compresión. Para identificar el caso y disparar una advertencia, se debe capturar el código de salida del último comando ejecutado, el comando ‘tar’, utilizando ‘$?’. Si el código de salida es 0, entonces fue exitoso. Si es 1 en cambio, indica que ocurrió una de estas inconsistencias, pero pudo realizarse la compresión. Otros códigos de salida indican errores que impiden el funcionamiento.
Automatización:
Cron es una herramienta que permite la ejecución periódica de instrucciones. Para agregar operaciones a realizar, una forma de hacerlo es creando un archivo dentro de /etc/cron.d/ con los comandos a ejecutar, definiendo el momento en que se quiera, mediante líneas con el siguiente formato: 
[Minuto] [Hora] [Día mes] [Mes] [Día semana] [usuario con que ejecuta] [comando]
Respecto al cronograma, si un valor no debe especificarse, el wildcard ‘*’ aplica para todos sus valores, entonces:
00 23 * * 1,3,5 root /bin/bash /opt/scripts/backup_full.sh /mnt/www_dir
Ejecuta el script de respaldo sobre el directorio /mnt/www_dir a las 23:00 los lunes, miércoles y viernes. Y:
00 00 * * * root /bin/bash /opt/scripts/backup_full.sh /var/logs
Respalda /var/logs todos los días a las 00:00
