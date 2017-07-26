# Installation d'un web server sous Mac

## Étape 1 : installation d'Homebrew

Installer les Command Line Tools

```bash
$ xcode-select --install
```   


Installer Homebrew
```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```   

Vérifier les conflits
```bash
$ brew doctor
```
Autoriser les liens entre les différents répertoires
```bash
$ brew tap homebrew/dupes
$ brew tap homebrew/versions
$ brew tap homebrew/php
$ brew tap homebrew/apache
```   
Si Homebrew est déja installé
```bash
$ brew update
```

## Étape 2 : installation d'Apache  
Afin d'éviter les conflits nous allons d'abord désactiver la version d'apache installer par défaut sur macOS et installer la dernière version de Apache via Homebrew.  

```bash
$ sudo apachectl stop
$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.apache.httpd.plist 2>/dev/null
$ brew install httpd24 --with-privileged-ports --with-http2
```

A la fin de l'installation le message suivant apparait
```bash
$ brew /usr/local/Cellar/httpd24/2.4.23_2: 212 files, 4.4M, built in 1 minute 45 seconds
```

Gardé de côté le chemin `/usr/local/Cellar/httpd24/2.4.25` nous allons en avoir besoin par la suite (le chemin peut changer suivant les versions)  

``` bash
$ sudo cp -v /usr/local/Cellar/httpd24/2.4.25/homebrew.mxcl.httpd24.plist /Library/LaunchDaemons
$ sudo chown -v root:wheel /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo chmod -v 644 /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
$ sudo launchctl load /Library/LaunchDaemons/homebrew.mxcl.httpd24.plist
```
Apache est maintenant bien installé, pour vérifier cela allé sur votre [localhost](http://localhost/)

**Quelques commandes utiles : **
```bash
$ sudo apachectl start
$ sudo apachectl stop
$ sudo apachectl -k restart
```

## Étape 3 : configuration d'Apache  

Par défaut Apache pointe vers le dossier `/Library/WebServer/Documents`, nous allons le faire pointer vers le dossier `Sites` dans lequel se trouve nos sites de développement

``` bash
$ open -e /usr/local/etc/apache2/2.4/httpd.conf
```
Chercher le terme `DocumentRoot` et changer le chemin par :

```bash
# Ancien chemin
DocumentRoot "/usr/local/var/www/htdocs"

# Nouveau chemin
DocumentRoot "/Users/your_user/Sites"

```
De même que le Directory
```bash
<Directory "/Users/your_user/Sites">
```

Quelques paramètres apache a activer:
```bash
#
# AllowOverride controls what directives may be placed in .htaccess files.
# It can be "All", "None", or any combination of the keywords:
# AllowOverride FileInfo AuthConfig Limit
#
AllowOverride All

# Décommenter
LoadModule rewrite_module libexec/mod_rewrite.so

# Les droits ps: pour connaitre le votre user taper 'whoami' dans le terminal
User your_user
Group staff

```

Redémarrer Apache pour que les modifications soient prises en compte

```bash
$ sudo apachectl -k restart  
```

## Étape 4 : installer PHP  

```bash
$ brew install php56 --with-httpd24
$ brew unlink php56
$ brew install php70 --with-httpd24
```

Les fichiers de configuration sont situés ici :

```
/usr/local/etc/php/5.6/php.ini
/usr/local/etc/php/7.0/php.ini
```

Il faut ensuite lier Apache et PHP en éditant la conf Apache

``` bash
$ open -e /usr/local/etc/apache2/2.4/httpd.conf
```

Les versions de PHP se sont ajoutées

```
LoadModule php5_module        /usr/local/Cellar/php56/5.6.26_3/libexec/apache2/libphp5.so
LoadModule php7_module        /usr/local/Cellar/php70/7.0.11_5/libexec/apache2/libphp7.so

```

Il faut les remplacer par :
```
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
```

Et ne garder qu'une version activée

```
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so
```

Il faut aussi autoriser l'exécution de PHP sur le répertoire `Sites`

```
<IfModule dir_module>
    DirectoryIndex index.php index.html
</IfModule>

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>

```
Redémarer Apache

```bash
$ sudo apachectl -k restart  
```

Pour vérifier que tout est correcte vous pouvez créer un fichier info.php à la racine du répertoire `Sites`

```php
<?php phpinfo();
```

Installation de librairies PHP :

```
# OPcache et APCu
$ brew install php56-opcache
$ brew install php56-apcu

# Xdebug
$ brew install php56-xdebug
```

Installation du script Switch

```bash
$ cd /usr/local/bin
$ curl -L https://gist.github.com/w00fz/142b6b19750ea6979137b963df959d11/raw > /usr/local/bin/sphp
$ chmod +x /usr/local/bin/sphp
```

Update du ~/.bash_profile  

```bash
$ sudo nano ~/.bash_profile

# Ajouter cette ligne
export PATH=/usr/local/bin:/usr/local/sbin:$PATH
```

Mise à jour du ~/.bash_profile  
```
source ~/.bash_profile
```

Editer le fichier de configuration Apache
```
$ open -e  /usr/local/etc/apache2/2.4/httpd.conf

# Remplacer ces lignes
LoadModule php5_module    /usr/local/opt/php56/libexec/apache2/libphp5.so
#LoadModule php7_module    /usr/local/opt/php70/libexec/apache2/libphp7.so

# Par

# Brew PHP LoadModule for `sphp` switcher
LoadModule php5_module /usr/local/lib/libphp5.so
#LoadModule php7_module /usr/local/lib/libphp7.so

```

On test ensuite le swich de version PHP
```
$ sphp 56
$ sphp 70
```

Connaitre la verison de PHP
```
$ php -v
```

Mise à jour de PHP et des package Homebrew

```
$ brew update

$ brew upgrade
```

## Étape 5 : installer MySQL

```bash
$ brew install mariadb
$ mysql_install_db
```

Démarer MySQL

```
$ mysql.server start
```

Pour démarer MySQL automatiquement

```
$ brew services start mariadb
```

Pour gérer vos bases de données avec une interface utilisateur utiliser [Sequel Pro](https://www.sequelpro.com/)

## Étape 5 : les Virtual Hosts d'Apaches

``` bash
$ open -e /usr/local/etc/apache2/2.4/httpd.conf
```

Décommenter

``` bash
LoadModule vhost_alias_module libexec/mod_vhost_alias.so

# Virtual hosts
Include /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

Gestions des Virtual hosts

```
$ open -e /usr/local/etc/apache2/2.4/extra/httpd-vhosts.conf
```

Exemple d'url pointant sur le dossier sites
```
<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites"
    ServerName localhost
</VirtualHost>

<VirtualHost *:80>
    DocumentRoot "/Users/your_user/Sites/mon-site"
    ServerName mon-site.dev
</VirtualHost>
```

Redémarer Apache

```bash
$ sudo apachectl -k restart  
```

Modifier votre fichier Host

```bash
sudo nano /etc/hosts
```

Ajouter votre nouvelle url

```bash
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1   mon-site.dev
```
Eviter les .local sous Mac Os ([lien](https://www.bram.us/2011/12/12/mamp-pro-slow-name-resolving-with-local-vhosts-in-lion-fix/))!

Pour automatiser cela nous allons utiliser Dnsmasq

```bash
$ brew install dnsmasq
```

Configurer pour que les url en '.dev' pointe vers le serveur local
```bash
$ echo 'address=/.dev/127.0.0.1' > /usr/local/etc/dnsmasq.conf
```
Démarage automatique de Dnsmasq
```bash
$ sudo brew services start dnsmasq
```

Ajouter un dossier de résolution DNS
```
$ sudo mkdir -v /etc/resolver
$ sudo bash -c 'echo "nameserver 127.0.0.1" > /etc/resolver/local'
```

Faire un test de ping : 'ping test.local'

> #### Sources
> [Homebrew](https://brew.sh/index_fr.html)  
> [macOS 10.12 Sierra Apache Setup: Multiple PHP Versions ](https://getgrav.org/blog/macos-sierra-apache-multiple-php-versions)  
>[macOS 10.12 Sierra Apache Setup: MySQL, APC & More...](https://getgrav.org/blog/macos-sierra-apache-mysql-vhost-apc)  
>[MariaDB](https://mariadb.com/kb/en/mariadb/installing-mariadb-on-macos-using-homebrew/)
>[MAMP Pro slow name resolving with .local vhosts in Lion (fix)](https://www.bram.us/2011/12/12/mamp-pro-slow-name-resolving-with-local-vhosts-in-lion-fix/)  
