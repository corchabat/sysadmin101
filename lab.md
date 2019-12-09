# Workshop: Sysadmin 101
## Básicos sobre administración de sistemas Linux
##### Por: Cecilia Ruiz :: ceciliaruiz@redhat.com


## 1. Instalación

## 2. Primeros pasos
      
### 2.1. Configuración de Red.
La primera configuración que llevaremos a cabo será asignar una dirección ip estática a nuestro sistema. Esto es especialmente necesario en el caso de los servidores que requieren ser accesados constantemente. La forma más directa de configurar la ip estática es a través del comando **nmcli** pero antes de eso, deberemos hacer algunas verificaciones:



Para conocer los dispositivos de red que están conectados a nuestro sistema, los listaremos con:

`$ sudo nmcli device show`

Y para mostrar las conexiones activas:

`$ sudo nmcli connection show`

Y ya que contamos con esta información, pasaremos a configurar la conexión con dirección IP estática:

`$ sudo nmcli connection add type ethernet ifname <nombre-del-dispositivo> con-name <nombre-de-la-conexión> ipv4.addresses <192.168.XX.XX>/24 ipv4.gateway <192.168.XX.XX> ipv4.method manual connection.autoconnect yes`

Verificamos de nuevo con: 

`$ sudo nmcli connection show`

### 2.2. Configurar Hostname.

Verificamos cuál es nuestro hostname acutal:

`$ echo $HOSTNAME`

Para modificar el hostname, utilizamos el comando **hostnamectl**

`$ hostnamectl set-hostname <nuevo-hostname>`

Confirmamos el cambio cerrando sesión con **ctrl+D** y volviendo a ingresar.

`$ echo $HOSTNAME`


### 2.3. Hacer upgrade/update.
Para obtener las últimas versiones de los paquetes que tenemos instalados.

`$ sudo yum -y update && yum -y upgrade`



### 2.4. Configurar SSH.

Verificamos si SSH está instalado y en qué versión se encuentra
         


### 2.5. Instalar y configurar Web Console.

`$ sudo yum install cockpit`

`$ sudo firewall-cmd --add-service=cockpit --permanent`

`$ sudo firewall-cmd --reload`

`$ sudo systemctl enable --now cockpit.socket`

En nuestro navegador, accedemos a

`https://hostname.example.com:9090` o `https://192.168.XX.XX:9090`

E ingresamos con nuestras credenciales estándar.







## 3. Usuarios, Grupos y Permisos.



## 4. Gestión del almacenamiento.




## 5. SELinux



## 6. Firewall



## 7. Remediación de vulnerabilidades



## 8. Ansible
