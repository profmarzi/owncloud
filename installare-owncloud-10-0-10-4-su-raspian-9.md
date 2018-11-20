# Istruzioni in italiano ricavate da un post di https://www.avoiderrors.com/owncloud-10-raspberry-pi-3-raspbian-stretch/
## Aggiornamento Raspberry
```
sudo su
apt update && apt upgrade
```
## Installazione LAMP Server
```
apt install apache2 -y
systemctl start apache2
systemctl enable apache2
```
## Installazione dipendenze ownCloud
```
apt install -y apache2 mariadb-server libapache2-mod-php7.0 \
    php7.0-gd php7.0-json php7.0-mysql php7.0-curl \
    php7.0-intl php7.0-mcrypt php-imagick \
    php7.0-zip php7.0-xml php7.0-mbstring
```
## Installazione Owncloud 10
In generale per installare pacchetti compressi si può usare una cartella di comodo per il download, effettuare l'installazione e poi cancellare il file compresso per risparmiare spazio. Tuttavia nel file system linux c'è la cartella /tmp che a ogni riavvio viene svuotata automaticamente e può convenire usare quella
```
cd /tmp
wget https://download.owncloud.org/community/owncloud-10.0.10.tar.bz2
tar -xvf owncloud-10.0.10.tar.bz2
chown -R www-data:www-data owncloud
```
Lo scompattatore creerà una cartella owncloud all'interno di /tmp. Questa cartella deve essere spostata nella cartella di riferimento del web server apache che è /var/www/html/
```
mv owncloud /var/www/html/
```
Ora bisogna creare un nuovo file di configurazione owncloud.conf che avvertirà apache quali sono le variabili di ambiente disponibili. Con l'editor si apre un nuovo file, io uso joe perché è identico a Word Star che usavo da piccolo.
```
sudo joe /etc/apache2/sites-available/owncloud.conf
```
e vi copio il codice seguente:
```
Alias /owncloud "/var/www/html/owncloud/"
<Directory /var/www/html/owncloud/>
 Options +FollowSymlinks
 AllowOverride All
<IfModule mod_dav.c>
 Dav off
 </IfModule>
SetEnv HOME /var/www/html/owncloud
SetEnv HTTP_HOME /var/www/html/owncloud
</Directory>
```
Con questa operazione si è solo inserito all'interno di /etc/apache2/sites-available il file di configurazione di owncloud. Perchè apache lo possa usare effettivamente occorrerebbe copiarlo anche nella cartella /etc/apache2/sites-enabled. Invece di copiarlo si può creare un link simbolico.
```
ln -s /etc/apache2/sites-available/owncloud.conf /etc/apache2/sites-enabled/owncloud.conf
```
Attivare anche i seguenti moduli:
```
a2enmod headers
systemctl restart apache2
a2enmod env
a2enmod dir
a2enmod mime
```
## Crare database MySQL
Prima di creare un database è necessario assegnare un utente con password
```
mysql -u root -p
```
Dopo aver inserito una password di propria scelta (non necessariamente la password del root di sistema) al prompt si scriveranno le seguenti righe, una alla volta, seguite da invio.
```
MariaDB [(none)]> create database owncloud;
MariaDB [(none)]> create user owncloud@localhost identified by '12345';
MariaDB [(none)]> grant all privileges on owncloud.* to owncloud@localhost identified by '12345';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit;
  ```
 Dopo il comando exit si riceverà un Bye e si esce dal prompt.
## Mountare HD esterno
## Abilitare certificati di sicurezza SSL
## Configurazione Owncloud 
