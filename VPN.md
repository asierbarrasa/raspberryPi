 	# Instalación de OpenVPN en una Raspberry Pi

# Como instalar OpenVPN en una raspeberry pi para navegar como si estuvieras en casa.

## A lo largo del siguiente documento se muestran los pasos a seguir para instalar y configurar de forma sencilla un servidor VPN en nuestra Raspeberry Pi. Desde la instalación de la VPN a la apertura y redirección de puertos necesaria para su funcionamiento.



# índice

1. [Raspberry](#id1)
   - [¿Qué es](#id2)
   - [Acceso mediante SSH](#id3)
     - [Cambiar contraseña](#id4)
2. [¿Qué es una VPN?](#id5)
	- [VPN vs Proxy](#id6)   
3. [Asignar IP estatica](#id7)  
	- [DHCP](#id8)  
4. [Redirección puertos](#id9)   
	- [DDNS (Acceso desde el esterior)](#id10)  
5. [Instalcion de OpenVPN](#id11)
6. [OpenVPN (TCP/UDP)](#id12)
7. [Generar nueva clave](#id13)
8. [Conclusiones](id14)



# Raspberry <a name="id1"></a>  

## ¿Qué es? <a name="id2"></a>  

Raspberry Pi es un **mini ordenador de pequeño tamaño, bajo coste y bajo consumo** cuyos
primeros modelos fueron lanzados en abril de 2012. El modelo utilizado para este tutorial es el
Raspberry Pi 3 Model B, en caso de usar el modelo 2, será necesario un módulo WIFI adicional, o
realizar la conexión mediante Ethernet. Por comodidad he realizado las pruebas mediante WIFI1
ya que el router no me es accesible de forma sencilla.

## Acceso mediante SSH <a name="id3"></a>  

Una conexión mediante SSH nos permitirá conectarnos de forma remota y cómoda a la raspberry.
Desde noviembre de 2016, el acceso mediante SSH viene desactivado por defecto, para activarlo
deberemos seguir los siguientes pasos:

1. Introduce `sudo raspi-config` en una terminal.
2. Selecciona `Interfacing Options`.
3. Selecciona `SSH`.
4. Elige `Yes`.
5. Selecciona `Ok`.
6. Pulsa `Finish`.



### Cambiar la contraseña <a name="id4"></a>  

Para cambiar la contraseña escriba `passwd `en una nueva terminal. Una vez conectado, primero escriba la contraseña por defecto: `raspberry` y a continuación la nueva contraseña.
Ya estamos listos para acceder a la raspberry de forma cómoda desde nuestro portátil, o móvil
mediante alguna app como `JuiceSSH`.

# ¿Qué es una VPN? <a name="id5"></a>  

## VPN vs Proxy, entendiendo las diferencias <a name="id6"></a>  

A pesar de que ambas se pueden utilizar para enmascarar una IP, es importante entender las
diferencias entre un servidor proxy y una VPN. La principal diferencia es que una **VPN**, cifra todo
el tráfico que se transmita desde tu dispositivo, por el contrario, un **servidor proxy**, tan solo
esconde la IP, haciendo de intermediario entre tu dispositivo y los servidores a los que se conecte
la aplicación.
Con un proxy actuando como intermediario en tu tráfico de Internet, toda la actividad que lleves
a cabo parecerá venir de otro lado.
Como hemos visto, aunque los dos realicen funciones similares, el uso de una VPN es si cabe más
interesante, ya que nos brinda más opciones de privacidad que él proxy.

# Asignar una IP estática <a name="id7"></a>  

> Si la raspberry esta conectada mediante cable, sustituir la interface wlanO por ethO. 

Con los siguientes comandos se asigna una IP estática y se establece el gateway por defecto.

`sudo nano /etc/dhcpcd.conf
interface wlan0
static ip_address=192.168.0.9
static routers=192.168.0.1
static domain_name_servers=192.168.0.1`

![1](https://user-images.githubusercontent.com/13755501/69046885-46a65000-09fa-11ea-9a18-302de644e05b.png)

## DHCP <a name="id8"></a>  

Es importante, cambiar la dirección inicial del servidor DHCP, ya que, si no, existe la posibilidad
que el servidor DHCP asigne la misma IP a dos dispositivos.
En este caso, como la IP estática asignada ha sido la 192.168.0.9, pondremos que el DHCP empiece
en la 192.168.0.10.

> Los menús de configuración pueden variar dependiendo del modelo y la marca del router.

![2](https://user-images.githubusercontent.com/13755501/69046886-46a65000-09fa-11ea-87f4-0a97ce446db5.png)	

# Redirección de puertos   <a name="id9"></a>  

En mi caso, está en el apartado del router “Agregar Servidor Virtual”.
1194, es el puerto que establece OpenVPN por defecto, dejaremos este, aunque se puede cambiar.

![3](https://user-images.githubusercontent.com/13755501/69046888-473ee680-09fa-11ea-8da8-a69cb95045a4.png)
![4](https://user-images.githubusercontent.com/13755501/69046889-473ee680-09fa-11ea-857f-17442270fc03.png)

## DDNS (Acceso desde el exterior) <a name="id10"></a>  

Ya que la IP publica de nuestra casa, es modificada por los operadores cada cierto tiempo, es
necesaria la utilización de un DDNS para poder acceder de forma remota sin la necesidad de
conocer la IP publica en todo momento. La DDNS se encarga de actualizar la IP asociada al
dominio con que el que nos hayamos registrado.

La solución a este problema fue NO-IP, su registro es gratuito, aunque cada 30 días te pide que
confirmes el dominio. Hay otras alternativas como DuckDNS, y FreeDns, aunque
personalmente nos las he probado.

`sudo apt-get update && sudo apt-get upgrade
wget http://www.no-ip.com/client/linux/noip-duc-linux.tar.gz
tar -zxvf noip-duc-linux.tar.gz
cd noip-2.1.9-1
sudo make
sudo make install`



![5](https://user-images.githubusercontent.com/13755501/69046890-473ee680-09fa-11ea-8a6b-247b20dbdfe2.png)

Hacemos que se inicie de forma automática al iniciar la raspberry.
Tras añadir noip2 a la carpeta `init.d`, lo añadimos a `update-rc.d`, que contiene los scripts que se
ejecutarán al iniciar la raspberry.

`sudo nano /etc/init.d/noip2
//Copiamos dentro lo siguiente
sudo /usr/local/bin/noip2
//Añadimos permisos
sudo chmod +x /etc/init.d/noip2
//Modificamos update-rc
sudo update-rc.d noip2 defaults
//Lanzamos noip
sudo /usr/local/bin/noip2`

# Instalacion de OpenVPN  <a name="id11"></a>  
`wget https://git.io/vpn -O openvpn-install.sh
sudo bash openvpn-install.sh`

![6](https://user-images.githubusercontent.com/13755501/69046891-47d77d00-09fa-11ea-81d7-d754d657bd24.png)

La **configuración** es muy sencilla, tan solo hay que:
1. Elegir la *dirección IP* que hayamos configurado previamente como estática, en nuestro
caso (192.168.0.9).
2. Escribir el *nombre* del servidor DDNS, que hayamos configurado en NO-IP.
3. Seleccionar el *protocolo*.
4. Elegir el *puerto*, en este caso dejaremos el puerto por defecto. Este habrá tenido que ser
el puerto que hayamos redireccionado previamente en el router.
5. Elegir la *DNS*, en mi caso, puse la de Google.
6. Elegir un *nombre* para el cliente.

# OpenVPN (TCP/UDP) <a name="id12"></a>  

A pesar de que la instalación por defecto de OpenVPN configura el protocolo como UDP, esta
opción se puede modificar para usar el protocolo TCP a partir de la versión 1.5 de openVPN.

Tras realizar alguna prueba, he de decir que a nivel practico, no he podido observar diferencia
alguna. Aunque tras leer en algunos [foros](https://www.bestvpn.com/guides/openvpn-tcp-vs-udp-difference-choose/), recomendaría por lo general el uso sobre **UDP**, salvo
que experimentes problemas de conexión.



# Generar una nueva clave <a name="id13"></a>  

Una vez instalado OpenVPN, para generar un nuevo usuario, con su correspondiente clave,
volvemos a lanzar el instalador.

`sudo bash openvpn-install.sh`



A continuación, pulsar `1`, y escribir el nuevo nombre que le daremos al usuario. Esto nos genera
un nuevo archivo que deberemos de sacar de las raspberry, por ejemplo, con el programa
WinSCP, y posteriormente instalar en el cliente.
Ya solo nos queda conectarnos.



![7](https://user-images.githubusercontent.com/13755501/69046892-47d77d00-09fa-11ea-9f18-06b6d64a8cba.png)
![8](https://user-images.githubusercontent.com/13755501/69046893-48701380-09fa-11ea-8ebf-959e16c69b0f.png)
![9](https://user-images.githubusercontent.com/13755501/69046894-48701380-09fa-11ea-9201-96e5c7ee6c4e.png)

# Conclusiones <a name="id14"></a>  

Aunque es cierto que hay más formas de realizar la instalación de OpenVPN, de forma más personalizada, generando de forma manual las credenciales y firmas, creo que esta es la forma mas cómoda, generando por defecto claves de una longitud segura, y permitiendo conectarnos de una forma sencilla, sin dependencias ni errores extraños en la instalación.
A continuación os dejo un [enlace](https://www.redeszone.net/redes/openvpn/?utm_source=related_posts&utm_medium=manual) a la guía de instalación mas completa que he encontrado. También podrían modificarse los parámetros del script de instalación utilizado: openvpn-install.sh para modificar, por ejemplo, las longitudes o algoritmos de las claves.
Si seguimos el tutorial paso a paso podremos, de forma sencilla y muy rápida tener una conexión segura allí donde vayamos, ya que OpenVPN cuenta tanto con aplicación para el móvil como para el ordenador.
Tras un par de semanas de uso, no he tenido ningún problema, cuelgue o bajada de velocidad mientras la he usado.
