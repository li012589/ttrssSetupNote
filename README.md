# Install TinyTinyRSS on debian

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
apt-get install php php-pgsql php-fpm php-curl php-cli php-apcu php-xml php-mbstring
```

Or (in debian 8)

```
apt install php5 php5-pgsql php5-fpm php-apc php5-curl php5-cli
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
chown -R www-data:www-data /var/www/html/tt-rss/
```

Navigate to _www.yourdomain.com/tt-rss/install_. Fill the form with database setup informations used before, notice to leave _hostname_ empty. The domain name can be left unchanged.

Then click "test configuration" and afterward "initialize". 

Copy the php script to _/var/www/html/tt-rss_ as _config.php_ as instructed.

## Setup feeds auto update

First, change _tt-rss_ folder's owner to _www-data_, so we will not be bothered by `Can't create lockfile` problem (**this is done before**).

```
chown -R www-data:www-data /var/www/html/tt-rss/
```

Then write daemon for auto update service at _/etc/systemd/system/ttrss-updater.service_

```
emacs /etc/systemd/system/ttrss-updater.service
```

Add following lines:

```
[Unit]
Description=ttrss_backend
After=network.target postgresql.service

[Service]
User=www-data
ExecStart=/var/www/html/tt-rss/update_daemon2.php

[Install]
WantedBy=multi-user.target
```

Then enable it by

```
systemctl start ttrss-updater
systemctl enable ttrss-updater
```

Logs can be  viewed by

```
journalctl -u ttrss-updater
```

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
cd /var/www/html/tt-rss
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

## Customize

Themes can be found at https://git.tt-rss.org/git/tt-rss/wiki/Themes

Basicly, themes can be installed by copy _*.css_ and folder with the same name to _/var/www/html/tt-rss/themes_ folder.

Also, plugins can be installed copy to _/var/www/html/tt-rss/plugins_.

The following settings are suggested:

| Setting                                  | Value      |
| ---------------------------------------- | ---------- |
| `Combined feed display`                  | `enabled`  |
| `Automatically expand articles in combined mode` | `disabled` |
| `Show content preview in headlines list` | `disabled` |

Or, `disabled combined mode, enabled widescreen mode` suggested.

## Reference

1. https://git.tt-rss.org/git/tt-rss/wiki/InstallationNotes
2. https://git.tt-rss.org/git/tt-rss/wiki/UpdatingFeeds
3. https://websetnet.com/host-rss-system-linux-tiny-tiny-rss/
4. https://www.digitalocean.com/community/tutorials/how-to-install-ttrss-with-nginx-for-debian-7-on-a-vps
5. https://www.linode.com/docs/web-servers/apache/host-your-own-rss-reader-with-tiny-tiny-rss-on-centos-7
6. https://hostpresto.com/community/tutorials/how-to-install-tiny-tiny-rss-on-centos-7/
7. https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts
8. https://github.com/naeramarth7/clean-greader