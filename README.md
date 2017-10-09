# Install TinyTinyRSS on debian8

\*all commands are assumed run by root

## Installing apache2

Run

```
apt-get update
apt-get install apache2
```

## Install PHP

Run

```
apt-get install php5 php5-pgsql php5-fpm php-apc php5-curl php5-cli
```

## Install PostgreSQL

Run

```
apt-get install postgresql
```

Configure it with

```
sudo -u postgres psql
postgres=# CREATE USER "www-data" WITH PASSWORD 'yourpasshere';
postgres=# CREATE DATABASE ttrss WITH OWNER "www-data";
postgres=# \quit
```

## Download TTRSS

Run

```
cd /var/www/html/
git clone https://tt-rss.org/git/tt-rss.git tt-rss
```

## Setting Up TT-RSS

First, repair permissions problems:

```
cd /var/www/html/
chmod -R 777 cache/images
chmod -R 777 cache/upload
chmod -R 777 cache/export
chmod -R 777 cache/js
chmod -R 777 feed-icons
chmod -R 777 lock
```

Navigate to _www.yourdomain.com/tt-rss/install_. Fill the form with database setup informations used before, notice to leave _hostname_ empty. The domain name can be left unchanged.

Then click "test configuration" and afterward "initialize". 

Copy the php script to _/var/www/html/tt-rss_ as _config.php_ as instructed.

## Setup virtual host

Edit virtual host configuration file

```
emacs /etc/apache2/sites-available/yourdomain.com.conf
```

Add lines like below:

```
<VirtualHost *:80>
    ServerAdmin admin@yourdomain.com
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/html/tt-rss
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Then run

```
a2ensite yourdomain.com.conf
service apache2 reload
```

Then configure _ttrss_ to use this domain

```
cd /var/www/html/
emacs ./config.php
```

Change this line 

```
define('SELF_URL_PATH', 'http://www.yourdomain.com/tt-rss');
```

to

```
define('SELF_URL_PATH', 'http://www.yourdomain.com/');
```

## Reference

1. https://git.tt-rss.org/git/tt-rss/wiki/InstallationNotes
2. https://websetnet.com/host-rss-system-linux-tiny-tiny-rss/
3. https://www.digitalocean.com/community/tutorials/how-to-install-ttrss-with-nginx-for-debian-7-on-a-vps
4. https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts