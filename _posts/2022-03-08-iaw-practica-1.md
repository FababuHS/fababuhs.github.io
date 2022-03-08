# IAW-PRACTICA-1
Álvaro Alejandro Fababú López  
Prática 1 del módulo de IAW  
2º ASIR - IES Celia Viñas

# Instalación de pila LAMP en una máquina virtual

El primer paso es asegurarnos de que el sistema está atualizado y al día

~~~
apt update 
apt upgrade -y
~~~

La opción *-y* permite introducir por parámetro la respuesta afirmativa a la pregunta que el sistema nos hará y automatizar la entrada de esta.

## Instalación servidor web Apache
~~~
apt install apache2 -y
~~~
## Instalación MySQL Server
~~~
apt install mysql-server -y
~~~
Cuanto termine la instalación de MySQL Server cambiamos la contraseña del usuario root a la que deseemos. En lugar de introducir la contraseña directamente vamos a asociar esta a una variable de forma en la que si la volvemos a necesitar en el futuro tengamos un atajo creado a ella.
~~~
MYSQL_ROOT_PASSWORD=root  
~~~
Ahora podemos cambiar la contraseña, sin la necesidad de dejarla escrita y visible en la consola
~~~
mysql <<< "ALTER USER root@'localhost' IDENTIFIED WITH mysql_native_password BY '$MYSQL_ROOT_PASSWORD';"
~~~
## Instalación de los paquetes de PHP
~~~
apt install php libapache2-mod-php php-mysql -y
~~~
Con esta instalación es necesario que reiniciemos el servidor Apache. Cuando se haya reiniciado copiamos el fichero info.php en el directorio de Apache.
~~~
systemctl restart apache2 
cp info.php /var/www/html
~~~
## Instalación de Adminer
Adminer es una herramienta de administración de bases de datos MySQL basada en php. Creamos un directorio para adminer dentro del directorio de Apache y nos traemos la última versión de Adminer desde su git.
~~~
cd /var/www/html
mkdir adminer
cd adminer
wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-mysql.php
mv adminer-4.8.1-mysql.php index.php
~~~
Actualizamos el propietario y el grupo del directorio /var/www/html
~~~
chown www-data:www-data /var/www/html -R
~~~
![Hola]({{ site.url }}/img/adminer.png)
## Instalación de phpMyAdmin
En primer lugar tenemos que instalar las dependencias necesarias para phpMyAdmin
~~~
apt install php-mbstring php-zip php-gd php-json php-curl -y
~~~
Ahora vamos a configurar las respuestas del instalador de phpMyAdmin para que podamos realizar una instalación desatendida
~~~
echo "phpmyadmin phpmyadmin/reconfigure-webserver multiselect apache2" | debconf-set-selections
echo "phpmyadmin phpmyadmin/dbconfig-install boolean true" | debconf-set-selections
echo "phpmyadmin phpmyadmin/mysql/app-pass password $PHPMYADMIN_APP_PASS" | debconf-set-selections
echo "phpmyadmin phpmyadmin/app-password-confirm password $PHPMYADMIN_APP_PASS" | debconf-set-selections

apt install phpmyadmin -y
~~~

## Instalación GoAccess
GoAccess es una aplicación web de análisis web de código abierto con interfaz visual.
~~~
echo "deb http://deb.goaccess.io/ $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/goaccess.list
wget -O - https://deb.goaccess.io/gnugpg.key | sudo apt-key add -
apt update
apt install goaccess -y
~~~
Una vez instalado creamos un subdirectorio en el directorio de Apache para los resultados del análisis de GoAccess y actualizamos el propietario y el grupo, y ejecutamos GoAccess.
~~~
mkdir /var/www/html/stats
chown www-data:www-data /var/www/html/stats -R

screen -dmL goaccess /var/log/apache2/access.log -o /var/www/html/stats/index.html --log-format=COMBINED --real-time-html
~~~
Queremos proteger el directorio del servidor con el análisis con usuario y contraseña. Creamos un par de variables para la dupla y protegemos el directorio.
~~~
STATS_USER=usuario  
STATS_PASSWORD=usuario 
htpasswd -cb /home/ubuntu/.htpasswd $STATS_USER $STATS_PASSWORD
~~~
Ahora copiamos el fichero de configuración de Apache en nuestro directorio y reiniciamos Apache
~~~
cd ~/iaw-practica-01
cp 000-default.conf /etc/apache2/sites-available

systemctl restart apache2 
~~~
![]({{ site.url }}/img/stats.png)
## Despliegue de la aplicación web
Para realizar el despliegue de la aplicación clonaremos en nuestro local el repo de José Juan que contiene el código fuente y el script para la base de datos.
~~~
cd /var/www/html

rm -rf /var/www/html/iaw-practica-lamp
git clone https://github.com/josejuansanchez/iaw-practica-lamp.git

mv iaw-practica-lamp/src/* /var/www/html

mysql -u root -p$MYSQL_ROOT_PASSWORD < iaw-practica-lamp/db/database.sql
~~~
Podemos eliminar el duplicado del fichero index como el directorio que contiene el repositorio. Actualizamos el propietario y grupo del directorio y reiniciamos Apache.
~~~
rm -f /var/www/html/index.html
rm -rf /var/www/html/iaw-practica-lamp
chown www-data:www-data /var/www/html -R
systemctl restart apache2
~~~
![]({{ site.url }}/img/app.png)
## Configuración certificado HTTPS
Queremos solicitar un certificado https para nuestra aplicación. Vamos a solicitarlo a Certbot que ofrece certificados https gratuitos. Para isntalarlo necesitamos instalar snap antes, que es aun asistente de instalación de paquetes.
~~~
snap install core 
snap refresh core 
~~~
Ahora eliminamos las instalaciones previas de Certbot para ejecuciones posteriores del script.
~~~
apt-get remove certbot
snap install --classic certbot
~~~
Y solicitamos el certificado. Crearemos dos variables más, una para el email ficticio que nos solicita y otra para el dominio gratuito que hemos creado en no-ip.
~~~
EMAIL_HTTPS=demo@demo.es  
DOMAIN=practicaiaw.ddns.net  
sudo certbot --apache -m $EMAIL_HTTPS --agree-tos --no-eff-email -d $DOMAIN
~~~