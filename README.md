# Deployment of the udacity-Item-Catalog

This is the documentation for deploying the [Udacity Item Catalog](https://github.com/elioschmutz/udacity_item_catalog#set-up-google-oauth2) properly.

<!-- MarkdownTOC levels="1" autolink=true autoanchor=false bracket="round" -->

- [Base overview](#base-overview)
- [Network overview](#network-overview)
- [Server-Configuration](#server-configuration)
    - [Updates](#updates)
    - [Disable ssh root-login](#disable-ssh-root-login)
    - [SSH Port](#ssh-port)
    - [Firewall](#firewall)
    - [Create a web user](#create-a-web-user)
    - [Apps Folder](#apps-folder)
- [DNS](#dns)
- [PostgresSQL](#postgressql)
- [Virtualenv](#virtualenv)
- [Webserver](#webserver)
- [Certbot](#certbot)
- [Install the Item-Catalog](#install-the-item-catalog)
- [Third-party resources used to complete this project](#third-party-resources-used-to-complete-this-project)

<!-- /MarkdownTOC -->

## Base overview

- URL: https://itemcatalog.elioschmutz.ch
- Github-URL: https://github.com/elioschmutz/udacity_item_catalog

- Type: AWS Lightsail Cloud Server
- IP: 18.185.100.161
- OS: Ubuntu 18.04.1 LTS
- SSH-Port: 2200

## Network overview
![Network diagram](./itemcatalog_network_diagram.png)

## Server-Configuration

The following steps are required to setup a secure server

### Updates

The server have to be up-to-date all the time. To do this, you should update your server regularly by executing:

```sh
sudo apt-get update
sudo apt-get upgrade
```

Use the `unattended-upgrades` package to configure automatic system updates.

```sh
sudo apt-get install unattended-upgrades
sudo dpkg-reconfigure --priority=low unattended-upgrades
```

See [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates) for more information about auto-updates with Ubuntu.

### Disable ssh root-login

Disabling the root-login over ssh is an important security setting because everyone knows the login-name of the root user. By disabling the root-login, an attacker have to guess valid usernames first.

1. make sure, you created another user within the sudoer group:

```sh
sudo adduser username
usermod -a -G sudo username
```

2. Edit the sshd config file:

```sh
sudo vim /etc/ssh/sshd_config
```

3. Locate the following linge

```sh
PermitRootLogin yes
```

and change it to:

```sh
PermitRootLogin no
```

4. Add the following line to only allow the newly created to login:

```sh
AllowUsers username
```

5. Save the changes and restart the service:

```sh
service ssh restart
```

See [Disabling ssh logins for root](https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root) for more information about disabling ssh logins.

### SSH Port

The default ssh-port is 22. Changing the default-ssh port will make the system more secure because an attacker have to guess or scan your ssh port first before he can start attacking your system.

1. Edit the sshd config file:

```sh
sudo vim /etc/ssh/sshd_config
```

2. Locate the following linge

```sh
Port 22
```

and change it to a new port:

```sh
Port 2200
```

5. Save the changes and restart the service:

```sh
service ssh restart
```

### Firewall

Only allow incoming requests on ports you really want to allow. To do this, we have to use a firewall.
Ubuntu comes with the UFW (uncomplicated firewall) which is a front-end to iptables.

1. Disallow all incomming requests

```sh
sudo ufw default deny incoming
```

2. Allow all outgoing requests

```sh
sudo ufw default allow outgoing
```

3. Open the ports you relly want. In our case, this would be the following ports:

- 80 (HTTP)
- 443 (HTTPS)
- 123 (NTP - network time protocol)
- 2200 (SSH !IMPORTANT: If we forget to open the ssh-port, we wouldn't be able to login again after activating the firewall)

```sh
sudo ufw allow http
sudo ufw allow https
sudo ufw allow ntp
sudo ufw allow 2200/tcp
```

4. Enable the firewall

```sh
sudo ufw enable
```

See [How to setup a firewall](https://www.digitalocean.com/community/tutorials/how-to-setup-a-firewall-with-ufw-on-an-ubuntu-and-debian-cloud-server) for more information about setting up the UFW on a ubuntu and debian server.

### Create a web user

The web-user is allowed to access all webapps-related content.

```sh
sudo adduser web
```

### Apps Folder

We provide all our apps within one place.

```sh
sudo mkdir /apps
```

and allow the web-user to read, write and execute the `/apps`-folder

```sh
chmod 775 /apps
chgrp web /apps
```

## DNS

Retgister a new Domain pointing to our IP-Address `18.185.100.161`.

## PostgresSQL

For the itemcatalog, we need a PostgresSQL Server.

1. Install postgresql

```sh
sudo apt install postgresql postgresql-contrib
```

2. Create a new postgres user:

```sh
sudo -u postgres createuser itemcatalog --interactive
```

3. And the related database for that user:

```sh
sudo -u postgres createdb itemcatalog
```

Your server have been setup and you should be able to connect to the database `itemcatalog` with the newly created postgesql-user `itemcatalog`. We'll use this setup while installing the item-catalog itself.

See [Setup and use Postgresql ](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04) for more information about setting up and using Postgresql

## Virtualenv

To get an isolated python for our itemcatalog, we have to use virtualenv.

1. Install pip
```sh
sudo apt-get install pip
```

2. Install virtualenv

```sh
sudo pip install virtualenv
```

## Webserver

We need an apache webserver to handle requests.

```sh
sudo apt-get install apache2
```

After installing it, you should be able to connect to `18.185.100.161` with a browser.
If you see the apache-demo page, the apache have been setup properly.

Add a new VHost for the Itemcatalog:

1. Create a new apache site:

```sh
sudo vim /etc/apache2/sites-available/001-item-catalog.conf
```

2. Configure the vhost with a wsgi-deamon process started with the `web` user:

```sh
<VirtualHost *:80>
        ServerName itemcatalog.elioschmutz.ch

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        WSGIDaemonProcess itemcatalog user=web group=web threads=5
        WSGIScriptAlias / /apps/udacity_item_catalog/application.wsgi

    <Directory /apps/udacity_item_catalog>
        WSGIProcessGroup itemcatalog
        WSGIApplicationGroup %{GLOBAL}
        Require all granted
    </Directory>

</VirtualHost>
```

3. Activate the site

```sh
ln -s /etc/apache2/sites-available/001-item-catalog.conf /etc/apache2/sites-enabled/001-item-catalog.conf
```

4. Restart the apache2 service

```sh
sudo service apache2 restart
```

## Certbot

If you want to provide your site over https, follow this steps to install an configure a "Let's encrypt" certificate:

[Create let's encrypt certificate](https://www.https-guide.de/was-ist-lets-encrypt/zertifikate-von-lets-encrypt-erstellen/)

## Install the Item-Catalog

1. Download the item-catalog:

```sh
cd /apps
git clone git@github.com:elioschmutz/udacity_item_catalog.git
```

2. Create a new virtualenv

```sh
cd udacity_item_catalog
virtualenv venv
```

3. Download the `activate_this.py` script:
```sh
wget https://raw.githubusercontent.com/pypa/virtualenv/master/virtualenv_embedded/activate_this.py aabb.py -P ./venv/bin
```

4. Create a `settings.cfg` file for production:
```sh
vim settings.cfg
```

with at least the following content:

```
SECRET_KEY="very secret random key"
DB_CONNECTION_STRING="postgresql://itemcatalog:your-password@localhost/itemcatalog"
```

5. Create oAuth credental-files:

Follow the steps in the readme of the [Udacity Item Catalog](https://github.com/elioschmutz/udacity_item_catalog#set-up-google-oauth2)

6. Restart apache to start the wsgi-server with the item-catalog.

Concratulation! You successfully setup the itemcatalog. If everything is configured correctly, you should be able to connect to `itemcatalog.elioschmutz.ch` and see the itemcatalog.

## Third-party resources used to complete this project

- [Flask Configuration](http://flask.pocoo.org/docs/1.0/config/)
- [Flask Deployment](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)
- [Load WSGI-Application as python module without installing it](https://stackoverflow.com/questions/38378560/my-wsgi-application-cannot-be-loaded-as-python-module-what-am-i-doing-wrong/38392475#38392475)
