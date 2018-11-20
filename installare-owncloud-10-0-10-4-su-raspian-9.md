# Istruzioni in italiano ricavate da un post di:
https://www.avoiderrors.com/owncloud-10-raspberry-pi-3-raspbian-stretch/
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
## Creare database MySQL
Prima di creare un database è necessario assegnare un utente con password. Ho scelto raspberry
```
mysql -u root -p
```
Dopo aver inserito una password di propria scelta (non necessariamente la password del root di sistema) al prompt si scriveranno le seguenti righe, una alla volta, ognuna seguita da invio.
```
MariaDB [(none)]> create database owncloud;
MariaDB [(none)]> create user owncloud@localhost identified by 'raspberry';
MariaDB [(none)]> grant all privileges on owncloud.* to owncloud@localhost identified by 'raspberry';
MariaDB [(none)]> flush privileges;
MariaDB [(none)]> exit;
  ```
 Dopo il comando exit si riceverà un Bye in uscita dal prompt.
 In pratica abbiamo impartito dei comandi che generano un nuovo database con le seguenti caratteristiche:
 Username: owncloud
 Password: raspberry
 Database: owncloud
 Server: localhost
## Montare HD esterno
Dovendo creare uno spazio per salvare i propri dati conviene collegare un HD al Raspberry. Questo serve a mettere a disposizione uno spazio maggiore ma soprattutto serve a non riempire e a logorare la memoria SD che contiene il sistema operativo. L'attuale versione 9 di Debian riconosce la presenza di un HD esterno e lo posiziona all'interno della cartella /media, tuttavia bisogna che ad ogni riavvio il sistema sappia che il nostro cloud si trova sull'HD e non altrove. Intanto l'HD deve essere formattato NTFS quindi colleghiamo l'HD ad una USB qualsiasi e lo formattiamo.
```
sudo apt-get install ntfs-3g -y
```
Successivamente creiamo una nuova cartella in /media. Diamo il nome ownclouddrive
```
sudo mkdir /media/ownclouddrive
```
Qusta cartella deve essere assegnata ad un proprietario che è il web server il cui nome, che abbiamo dià visto prima è www-data.Conviene creare un gruppo www-data in quanto ora sono diverse le risorse che gli appartengono: quella creata in /var/www/html/owncloud e ora /media/ownclouddrive.
```
sudo groupadd www-data
sudo usermod -a -G www-data www-data
```
e vi aggiungiamo l'ultimo arrivato
```
sudo chown -R www-data:www-data /media/ownclouddrive
sudo chmod -R 775 /media/ownclouddrive
```
La seconda riga di comando serve ad abilitare la scrittura nella cartella
Siccome l'HD può essere inserito in uno degli ingressi USB che può essere diverso di volta in volta in caso si smontaggio del cloud,il sistema si serve di una tabella, fstab, dove individua le caratteristiche dei dispositivi collegati. Ad ogni dispositivo corrispondono dei connotati univoci.Le seguenti istruzioni servono a questo:
```
id -g www-data
id -u www-data
```
Ora serve identificare il numero di serie dell'HD, l'UUID
```
ls -l /dev/disk/by-uuid
```
![image](https://user-images.githubusercontent.com/9042964/48806214-ec58ad80-ed19-11e8-9ffa-f8d688c346ab.png)
Per i primi due comandi il sistema risponde con il numero 33 mentre il terzo comando evidenzia l'assenza di un HD. Se lo inseriamo e ripetiamo il terzo comando otteniamo:
![image](https://user-images.githubusercontent.com/9042964/48806540-fcbd5800-ed1a-11e8-8493-63a36a89c1bf.png)
## Abilitare certificati di sicurezza SSL
## Configurazione Owncloud 

