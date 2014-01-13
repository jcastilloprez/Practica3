Práctica 3: Diseño de máquinas virtuales
=========

**Diseñar una máquina virtual para una aplicación específica.**

## Introducción 

En esta práctica lo que se pide es realizar una comparación de distintos sistemas operativos, con distintas 
configuraciones y ver cuál de ellos nos ofrece un rendimiento óptimo. Para ello se crearán distintas máquina virtuales que
cumplan estas características. 

Para realizar esta práctica voy a utilizar como software de virtualización el programa VMware Player y en total voy a 
crear 6 máquinas virtuales. Estas máquinas van a ser 3 de Ubuntu Server, en concreto la versión 12.04.3, y 3 de CentOS, en
la versión 6.5. De cada trio de máquinas, 2 máquinas van a actuar como máquinas servidoras de mi aplicación y la tercera 
va a actuar como máquina balanceadora, para repartir la carga de trabajo entre ambas máquinas servidoras. 

## Máquinas virtuales

Cada trio de máquinas van a tener las siguientes características:

* La primera máquina servidora va a disponer de: 1 procesador, 1024 MB de memoria RAM y el servidor LAMP. 
* La segunda máquina servidora va a disponer de: 1 procesador, 512 MB de memoria RAM y el servidor LAMP.
* La tercera máquina, que es la máquina balanceadora, va a disponer de: 1 procesador, 1024 MB de memoria RAM y como 
sistema balanceador, va a disponer del balanceador `nginx`. 

Empezamos por instalar el balanceador `nginx` en la máquina balanceadora. Para ello, lo primero que tenemos que hacer es
importar la clave del repositorio de software:

> ```
> wget http://nginx.org/keys/nginx_signing.key
> apt-key add nginx_signing.key
> rm -f nginx_signing.key
> ```

A continuación, debemos añadir el repositorio, editando el fichero **/etc/apt/sources.list** y añadiendo al final las 
siguientes líneas:

> ```
> echo "deb http://nginx.org/packages/ubuntu lucid nginx" >> /etc/apt/sources.list
> echo "deb-src http://nginx.org/packages/ubuntu lucid nginx" >> /etc/apt/sources.list
> ```

Ahora ya podemos instalar `nginx`:

> ```
> apt-get update
> apt-get install nginx
> ```

Una vez que se instale, procedemos a su configuración como balanceador de carga de las máquinas balanceadoras. Para ello
nos desplazamos al fichero de configuración de `nginx` que se encuentra en `/etc/nginx/conf.d/default.conf` y lo 
rellenamos tal y como se muestra a continuación: 

![Práctica 3 - Foto 1](http://ubuntuone.com/5395vgrA58WUcrxAg9rmfF)

Ahora al final le añadimos las direcciones IP de las máquinas que van a formar parte del balanceo de carga: 

![Práctica 3 - Foto 2](http://ubuntuone.com/1OJpXyKoS4oR9zvmoU47Wm)

En dicha última configuración, hemos tenido en cuenta las características de cada una de las máquinas servidoras. Es 
decir, a la máquina más potente, que es la máquina que tiene 1024 MB de memoria RAM, le hemos puesto que realice el 
doble de peticiones que la otra máquina menos potente que tiene 512 MB de memoria RAM. Esto lo hacemos para evitar que 
la máquina menos potente se venga abajo con una sobrecarga de trabajo y así tener más consistencia en nuestro sistema 
web.

Una vez que hemos rellenado correctamente el fichero de configuración de `nginx`, reiniciamos el servicio para que 
obtenga estos últimos cambios que le hemos realizado en su fichero de configuración: 

![Práctica 3 - Foto 3](http://ubuntuone.com/1XhdRkWbKKsRplLmN3DNey)

Para demostrar que hemos realizado bien la configuración del balanceador y que cada vez que lanzemos peticiones al 
balanceador, este nos reparte las peticiones entre las dos máquinas servidoras, vamos a hacer uso del comando cURL. Pero
antes y para demostrar qué máquina es la que realiza la petición, vamos a modificar la página inicial que trae Apache 
por defecto de ambas máquinas.

Una vez que hemos realizado la modificación a los ficheros por defecto de Apache, nos vamos a la máquina anfitriona y 
con el comando cURL lanzamos peticiones a la máquina balanceadora:

![Práctica 3 - Foto 4](http://ubuntuone.com/3CPaPfklj2HgPSXOmrXzHd)

y vemos como esta nos reparte la carga entre ambas máquinas servidoras, realizando la máquina más potente el doble de 
peticiones que la máquina menos potente. 

Ahora con el trio de máquinas de CentOS debemos realizar los mismos pasos: instalar el servidor LAMP en las máquinas 
servidoras, instalar y configurar el balanceador `nginx` en la máquina balanceadora y probar que funcione. 

Instalar el servidor LAMP en CentOS debe ser manualmente y no como en Ubuntu Server que mientras creabamos la máquina 
esta nos permitia la opción de instalar dicho servidor. Por tanto, para instalar LAMP en CentOS debemos instalar primero
el servidor Apache con:

`yum install httpd httpd-devel`

y una vez que se instale lo lanzamos: 

![Práctica 3 - Foto 5](http://ubuntuone.com/2AMJ9xw6YNrB27PVpdUnLx)

Ahora instalamos la base de datos MySQL con la orden: 

`yum install mysql mysql-server mysql-devel`

y lo arrancamos: 

![Práctica 3 - Foto 6](http://ubuntuone.com/5Fb9chpTRGQkQYeXEx1lW6)

Por último instalamos PHP con todas sus dependencias para que enlace correctamente con Apache y MySQL. Para ello 
utilizamos la siguiente orden: 

`yum install php php-mysql php-common php-gd php-mbstring php-mcrypt php-devel php-xml`

y reiniciamos Apache para que recoja todos los cambios actuales: 

![Práctica 3 - Foto 7](http://ubuntuone.com/2dzIehoLwvUvXRzAsiHFv5)

El servidor LAMP lo instalamos en las dos máquinas servidoras y en la máquina balanceadora instalamos el balanceador 
`nginx`. Para ello primeramente instalamos los repositorios que son requeridos para que nginx pueda ser instalado: 

![Práctica 3 - Foto 8](http://ubuntuone.com/6p77HTduhyzKnaZCgnx9Fy)

e instalamos `nginx` con:

`yum install nginx`

Una vez que termine de instalarse lo iniciamos: 

![Práctica 3 - Foto 9](http://ubuntuone.com/1dAAGf2aaREfIutecD2Sge)

## Aplicación

La aplicación que voy a meter en cada una de las máquinas servidoras, va a ser una aplicación que utiliza el servicio de
Google Maps. Dicha aplicación nos pide que introduzcamos un lugar y a continuación nos muestra el mapa de dicho lugar. 
Ahora con ese lugar podemos hacer: aumentar para ver más detalles del lugar, igualmente disminuir, insertar marcadores 
en el mapa, utilizar Street View y ver las coordenadas geográficas del lugar exacto en donde nos encontramos. 

El fichero con la aplicación se encuentra en nuestra máquina anfitriona y debemos pasar dicho fichero a cada una de las 
máquinas servidoras. Para CentOS es muy fácil ya que al disponer de entorno gráfico, podemos meter el fichero en un pen 
y directamente sustituir el fichero que trae Apache por defecto por este otro. 

Abrimos un navegador, lanzamos una petición a la máquina balanceadora y podemos ver que nuestra aplicación aparece en el
navegador: 

**Captura 10**

En caso contrario, en Ubuntu Server al no tener entorno gráfico debemos de buscar otra alternativa. Dicha alternativa 
pasa por compartir una carpeta de nuestra máquina anfitriona con las máquinas servidoras. Para realizar esto nos vamos 
al apartado configuración de nuestra máquina virtual y en la pestaña "Options" elegimos la opción "Shared Folders" y 
añadimos la carpeta que queremos compartir con la máquina virtual. 

![Práctica 3 - Foto 11](http://ubuntuone.com/2HhWHjBxeQLhFjN2CEO6sP)

A continuación iniciamos la máquina, instalamos los tools y realizamos los siguientes pasos: 

> ```
> mkdir -p /media/cdrom
> mount /dev/cdrom /media/cdrom
> cd /media/cdrom
> cp VM*.tar.gz /tmp
> apt-get -y install linux-headers-server build-essential
> cd /tmp
> umount /media/cdrom
> tar xzvf VM*.tar.gz
> cd vmware-tools-distrib
> mkdir -p /usr/lib64
> ./vmware-install.pl -d
> reboot
> cd /mnt/hgfs
> ```

y en este último directorio nos aparece la carpeta que estamos compartiendo con el fichero de la aplicación dentro. Ya 
solo nos queda sustituir el fichero que trae Apache por defecto por este otro con la siguiente orden: 

`cp index.html /var/www/`

Realizamos la operación con ambas máquinas, abrimos un navegador y lanzamos una petición a la máquina balanceadora:

![Práctica 3 - Foto 12](http://ubuntuone.com/0MJwqtO4IbK4eqKc6N3iOV)
