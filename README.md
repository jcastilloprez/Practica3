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

Para demostrar que hemos realizado bien la configuración del balanceador y que cada vez que lanzemos peticiones al balanceador, este nos reparte las peticiones entre las dos máquinas servidoras
