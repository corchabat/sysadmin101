# Workshop: Sysadmin 101
## Básicos sobre administración de sistemas Linux
##### Por: Cecilia Ruiz :: ceciliaruiz@redhat.com


## 1. Instalación

## 2. Primeros pasos

### 2.0.1 Uso de man y journalctl
El comando man nos es de mucha ayuda al momento de trabajar con un comando cuyas opciones no conocemos de memoria:

`$ man journalctl`

Revise la salida de este comando: la descripción de las opciones para el comando y los ejemplos de utilización al final del manual. Para buscar una palabra en específico, dentro del manpage, podemos ingresar */palabra* y avanzar a la siguiente coincidencia con la tecla *n* (next).

journalctl es una herramienta muy importante y muy útil para trabajar con logs, sin embargo, es necesario saber filtrar sus resultados para optimizar su uso. Si nosotros ingresamos:

`$ sudo journalctl`

Obtendremos como salida una lista interminable de logs. Por otro lado, si nosotros ingresamos:

`$ sudo journalctl -u httpd.service`

Únicamente obtendremos los logs relacionados con el servicio httpd.

Para filtrar una ventana de tiempo, podemos ejecutar el siguiente comando:

`$ sudo journalctl -since "2020-04-22 10:30:00" --until "2020-04-22 10:45:00"`

O bien, para monitorear los logs conforme se van generando, podemos ejecutar:

`$ sudo journalctl -ef`

### 2.0.2 Edición de texto desde línea de comando.
Es posible trabajar con archivos de texto directamente desde línea de comando. Vi es el editor de texto por defecto en unix y linux. Es importante tomar en cuenta que Vi tiene dos modos de trabajo: modo de edición y modo de comando. Para comenzar a utilizarlo, necesitamos conocer algunas instrucciones básicas:

i -    pasar a modo de edición
esc -  pasar a modo comando
x -    borrar un caracter
dd -   borrar línea completa
:w -   guardar
:wq -  guardar y salir
:q! -  salir sin guardar cambios
:u -   deshacer

Para probar Vi, vamos a generar un archivo que tenga como contenido parte de alguna canción:

`$ vi cielito`

Presionamos la tecla *i* para comenzar a insertar/editar:

      De la sierra morena, Cielito Lindo, vienen bajando
      Un par de ojitos negros, Cielito Lindo, de contrabando
      Ay, ay, ay, ay! Canta y no llores
      Porque cantando se alegran, Cielito Lindo, los corazones
      
Para guardar cambios y salir, presionamos la tecla *esc*, que nos manda de vuelta al modo de comando, donde ejecutaremos *:wq* para guardar y salir.
Tome este archivo y haga pruebas con él: pruebe eliminar una línea entera, deshacer cambios o copiar y pegar.
      
### 2.1. Configuración de Red.
La primera configuración que llevaremos a cabo será asignar una dirección ip estática a nuestro sistema. Esto es especialmente necesario en el caso de los servidores que requieren ser accesados constantemente. La forma más directa de configurar la ip estática es a través del comando **nmcli** pero antes de eso, deberemos hacer algunas verificaciones:

Para conocer los dispositivos de red que están conectados a nuestro sistema, los listaremos con:

`$ ip link show`

Observamos que la única interfaz que lista es *ens3* y que se encuentra en estado DOWN, lo cual quiere decir que la interfaz se encuentra deshabilitada. Vamos a habilitarla:

`$ sudo ip link set ens3 up`

Para mostrar un resumen de qué dispositivos de red tenemos en nuestra máquina, si están habilitados y si tienen asignada una conexión, utilizamos el comando *nmcli*:

`$ nmcli device s`

Y para mostrar qué conexiones activas tenemos:

`$ nmcli connection show`

Ya que contamos con esta información, pasaremos a configurar la conexión con dirección IP estática:

`$ nmcli connection add type ethernet ifname <nombre-del-dispositivo> con-name <nombre-de-la-conexión> ipv4.addresses <192.168.XX.XX>/24 ipv4.gateway <192.168.XX.XX> ipv4.dns <192.168.XX.XX> ipv4.method manual`

Considerar los siguientes parámetros:
ifname: ens3
con-name: ens3
ip4: 10.0.0.12/24
gw: 10.0.0.1
dns: 10.0.0.2

Verificamos de nuevo con: 

`$ sudo nmcli connection show`

`$ ping node3`

`$ ip route show`

`$ netstat -tulpn`

### 2.2. Habilitar firewall para un puerto o servicio.

Un firewall sirve para proteger nuestros sistemas del tráfico de red no deseado a través de reglas. En RHEL 8, el firewall se maneja como un servicio llamado firewalld y las reglas se definen con la utilidad de línea de comando firewall-cmd.

`$ sudo firewall-cmd --permanent --add-port=80/tcp`
`$ sudo firewall-cmd --permanent --add-service=http`
`$ sudo firewall-cmd --reload`


### 2.3. Hacer upgrade/update.
Para obtener las últimas versiones de los paquetes que tenemos instalados.

`$ sudo yum -y update && yum -y upgrade`

Para instalar un paquete, también se utiliza el comando yum:

`$ yum -y install bash-completion`

### 2.4. Comandos básicos de navegación entre filesystems.

Al navegar entre directorios vía línea de comando, nos encontramos con algunos comandos que simplifican esta navegación o nos permiten generar, eliminar, copiar o mover los archivos:

Por ejemplo, el comando *pwd* nos muestra el "present working directory", o bien, la ruta completa del directorio donde nos encontramos actualmente:
`
$ pwd`

De tal manera que si necesitamos cambiar de directorio, podemos utilizar el comando *cd* o "change directory":

`$ cd /home/student`

Ahora bien, si el directorio que necesitamos no existe, podemos crearlo com *mkdir":

`$ mkdir /home/student/dir1`

O si vamos a crear directorios anidados, añadimos la opción *-p*

`$ mkdir -p /home/student/dir1/dir2/dir3`

Para observar qué archivos y directorios están contenidos dentro de cierta ruta, tenemos el comando *ls*, por ejemplo, si ejecutamos *ls -las*:

`$ ls -las /home/student`

Podremos ver información acerca de quienes son los propietarios, timestamps y tamaños de todo lo que se encuentra dentro de /home/student.

### 2.5. Iniciar, detener y habilitar servicios.

En RHEL 8,los servicios se gestionan a través de la herramienta de línea de comando *systemctl*:
Estado actual de un servicio
`$ systemctl status firewalld`
Iniciar un servicio
`$ sudo systemctl start firewalld`
Habilitar un servicio
`$ sudo systemctl enable firewalld`
Detener un servicio
`$ sudo systemctl stop firewalld`


## 3. Usuarios, Grupos y Permisos.

En esta sección, trabajaremos con los básicos de manejo de usuarios y grupos locales, así como de permisos de archivos y carpetas.

### 3.1. Gestión de usuarios y grupos.

Primero, mostraremos información acerca del usuario *sysadmin* utilizando el comando *id*:

`$ id`

Podemos observar que el usuario tiene un user id (uid), que es en realidad el identificador del usuario, al igual que un group id (gid).

Dentro del archivo */etc/passwd* se encuentran listados todos los usuarios locales de nuestro sistema junto con su uid, gid, la ruta de su directorio home (si es que cuenta con uno) y su shell. Observemos su contenido:

`$ cat /etc/passwd`

De manera análoga, tenemos el archivo */etc/group* con el detalle de los grupos locales y qué usuarios pertenecen a ellos. Observemos su contenido:

`$ cat /etc/group`


Para agregar nuevos usuarios y grupos existen los comandos *useradd* y *groupadd*. Para probar esto, vamos a generar dos usuarios adicionales: sysadmin1 y sysadmin2.

`$ sudo useradd sysadmin1`

`$ sudo useradd sysadmin2`


Para poder iniciar sesión como estos usuarios, hay que asignarles una contraseña con el comando *passwd*:

`$ sudo passwd sysadmin1`

`$ sudo passwd sysadmin2`


Una manera de asegurar que las contraseñas sean privadas y seguras es a través del comando *chage* que nos permite modificar nuestras políticas de contraseñas.

Para mostrar la política de contraseña actual para un usuario:

`$ sudo chage -l sysadmin1`

Para asegurarnos de que el usuario cambie su contraseña cada 60 días como máximo: 

`$ sudo chage -M 60 sysadmin1`

O bien, para asegurarnos de que el usuario tenga que cambiar su contraseña tras iniciar sesión por primera vez:

`$ sudo chage -d 0 sysadmin1`

Probemos iniciar sesión como sysadmin1 para confirmar este último cambio.

`$ su - sysadmin1`

Para crear grupos, utilizamos el comando *groupadd* de forma muy similar a *useradd*:

`$ sudo groupadd admin`


Para agregar usuarios a este grupo, utilizaremos el comando *usermod* que nos permite modificar cuentas de usuario:

`$ sudo usermod sysadmin1 -G admin`

`$ sudo usermod sysadmin2 -G admin`



### 3.2. Permisos y pertenencia de archivos y carpetas

Podemos listar los permisos y pertenencia actuales de archivos y carpetas con el comando *ls -l*

`$ ls -l`

Para cambiar esto, utilizamos el comando *chmod* en el caso de permisos y *chown* para la pertenencia. Generemos archivos vacíos dentro de un directorio compartido para probar esto:

`$ mkdir /tmp/share`

`$ cd /tmp/share`

`$ touch file1.txt file2.txt file3.txt`

Vamos a modificar los permisos para file1 con el comando *chmod*, para que pueda ser leído, editado y ejecutado por el usuario a quien pertenece; leído y editado por el grupo al que pertenece y únicamente leído por cualquier otro usuario:

`$ sudo chmod 764 file1.txt`

Corroboramos el cambio con:

`$ ls -l`

Vamos a probar con otra sintaxis para modificar los permisos del archivo file2:
 
`$ chmod u+rwx g+rw o+r file2.txt`
 
 Verificamos que los archivos tienen los mismos permisos.
 
Ahora veamos cómo se modifica la pertenencia con el comando *chown*, asignando la de file1 al usuario sysadmin2 y al grupo admin y la de file2 al usuario sysadmin3 y al grupo sysadmin3:
 
`$ sudo chown sysadmin2:admin file1.txt`

`$ sudo chown sysadmin3:syadmin3 file2.txt`


## 4. Gestión del almacenamiento.
En esta sección, entenderemos qué es el Logical Volumen Manager (LVM), cómo podemos utilizarlo para gestionar el almacenamiento de dispositivos de bloques y cómo podemos disponer de este espacio.

En primer lugar, necesitamos conocer qué dispositivos de bloque están conectados a nuestra máquina. Para esto, podemos ayudarnos del comando *lsblk*:

`$ lsblk`

Podemos observar que se especifican las rutas de los dispositivos, su tamaño y si están montados y dónde. En este caso, notamos los discos */dev/vdb* y */dev/vdc*, ambos de 5GB y que no han sido montados. Estos son los discos con los que trabajaremos.

Para gestionar el espacio de estos dispositivos utilizando LVM, es necesario declararlos como Physical Volumes (PV's). LVM segmenta estos PV's en unidades llamadas extents, que son los bloques más pequeños que constituyen este Physical Volume (volúmen físico). Esto lo haremos con el comando *pvcreate*:

`$ sudo pvcreate /dev/vdb /dev/vdc`

Listamos los Physical Volumes existentes con el comando *pvs* y mostramos más detalles con el comando *pvdisplay*:

`$ sudo pvs`

`$ sudo pvdisplay /dev/vdb`

Ahora, vamos a generar un Volume Group (VG) a partir de estos PV's, siendo el VG un pool de almacenamiento compuesto por uno o más volúmentes físicos y del cual podemos generar múltiples Volúmenes Lógicos para consumir dicho espacio. Esto lo haremos con el comando *vgcreate*

`$ sudo vgcreate vg-workshop /dev/vdb /dev/vdc`

Del mismo modo, listamos los Volume Groups existentes  con el comando *vgs* y mostramos más detalles con el comando *vgdisplay*:

`$ sudo vgs`

`$ sudo vgdisplay vg-workshop`

Una vez creado el VG, generaremos a partir de éste los Logical Volumes (LV's) que utilizaremos para poder disponer del espacio. Notemos que el espacio disponible en nuestro VG es de 10 GB. Vamos a crear un LV de 6GB de tamaño utilizando el comando *lvcreate*:

`$ sudo lvcreate -L 6G -n lv-workshop vg-workshop`

Igualmente, listamos los Logical Volumes existentes  con el comando *lvs* y mostramos más detalles con el comando *lvdisplay*:

`$ sudo lvs`

`$ sudo lvdisplay lv-workshop`

Antes de poder disponer de este espacio, es necesario otorgarle un filesystem, es decir una estructura de cómo se almacenan datos y cómo se dispone de ellos. El filesystem que utilizaremos es ext4, ya que es el estándar para Linux debido a su estabilidad y rapidez. Asignamos filesystem a un LV utilizando el comando *mkfs*:

`$ sudo mkfs.ext4 /dev/vg-workshop/lv-workshop`


Por último, para darle uso a este filesystem de 6GB, vamos a montar el LV. Para esto, vamos a crear un directorio utilizando *mkdir* y a montar en él nuesto fs utilizando el comando *mount*.

`$ mkdir /home/student/mnt-workshop`

`$ sudo mount /dev/vg-worksop/lv-workshop /home/student/mnt-workshop`

El comando *df -h* nos muestra todos los puntos de montaje que tenemos en nuestra máquina. Corroboremos que el montaje y tamaño de nuestro filesystem son correctos:

`$ df -h`


## 5. Herramientas de Optimización

### 5.1 Instalar y configurar Web Console.

En "node2" realizamos los siguientes pasos:

`$ sudo yum install cockpit`

`$ sudo firewall-cmd --add-service=cockpit --permanent`

`$ sudo firewall-cmd --reload`

`$ sudo systemctl enable --now cockpit.socket`

En el navegador de workstation, accedemos a

`https://hostname.example.com:9090` o `https://10.0.0.12:9090`

E ingresamos con nuestras credenciales estándar.


