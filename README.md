# udacity-linux-server-configuration

## The Live site can be found here:[live catalog site](http://ec2-13-210-20-118.ap-southeast-2.compute.amazonaws.com)
## Server Details:
>> + ip address:13.210.20.118
>> + url:ec2-13-210-20-118.ap-southeast-2.compute.amazonaws.com

# Configuring a Linux Server
### This is the fifth and final Project of Udacity Full Stack Nanodegree program
### The project requires:
+ Setting up a cloud linux instance,
>> Amazon lightsail is recommended with a ubuntu image os only
+ Update all currently installed packages.
+ Setting up a user `grader` with sudo privileges
+ Setting up key-based authentication for the user `grader`
+ Disabling root login
+ Disabling password based login
## Firewall Configuration
>> + Disabling all ports for ssh except 2200
>> + Disabling all incoming requests except for ssh at port 2200 and http at port 80 and for ntp at port 123
>> + Enable all outgoing
## Database Configuration
>> + Install postgresql
>> + Do not allow remote connections

>> + Configure postgresql for the web app to be hosted
## WebApp
>> + The project requires catalog app to be hosted on instance which is the third app for udacity degree.
The project had python backend and had sqllite database originally.
>> + The sqlite is to be replaced with postgres.
>> + Configure the local timezone to UTC.
>> + Install `git` and clone the respository found [here](https://github.com/Abhi-sigma/catalog)
## Setting up a Apache Server
>> + Install and setup a Apache Server to host the catalog app.

# Steps Taken to Fulfill Requirements

### Setting up Linux instance
>> + Signup or signin to [amazon](https://lightsail.aws.amazon.com/)
>> + Follow the instructions  and choose a linux image
>> + Download or create your own ssh key pair.
>> + Save the private key if you have downloaded it in a location easy to access.Your ssh client
may have specific instructions in using the key so follow the instructions.
>> + You can also ssh from the browser based client provided with the instance image in the browser.

## Set up a `grader` user
>> + ssh into terminal
>> + `sudo adduser grader `
 >> + follow the instructions
>> ### Give grader sudoer privilges
>>>> + `sudo touch/etc/sudoers.d/grader`
>>>> + `sudo nano /etc/sudoers.d/grader`
>>>> + type in `grader ALL=(ALL:ALL) ALL`,save and quit
### Add ssh keys for user `grader`
>> + generate a ssh key pair using your local terminal
>> + switch user `su - grader`.Enter password.
>> +  `nano .ssh/authorized_keys`
>> + copy the public key(generally with pub at end) and paste in the editor,save and quit.
>> + change permissions
>>>> + $ chmod 700 .ssh
>>>> + $ chmod 644 .ssh/authorized_keys
>> + reload ssh `service ssh restart`
>> + logout and try using ssh as a grader with the generated private key.

## Update all currently installed packages
>> + sudo apt-get update
>> + sudo apt-get upgrade

## Change the SSH port from 22 to 2200
>> + `sudo nano /etc/ssh/sshd_config`
>> + Change port from 22 to 2200.
>> + Reload using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)
#### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
>> + `sudo ufw allow 2200/tcp`
>> + `sudo ufw allow 80/tcp`
>> + `sudo ufw allow 123/udp`
>> + `sudo ufw enable`

### Note:If you are using Lightsail,please enable custom connection on port 2200 from the browser based console.This really
gave me big headache in the project.So,before u enable firewall,please change this setting.

## Configure the local timezone to UTC
>> + Configure the time zone `sudo dpkg-reconfigure tzdata`.

## Install Apache and mod_wsgi interface to serve a Python mod_wsgi application
>> + Install Apache `sudo apt-get install apache2`
>> + Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`

## Install and configure PostgreSQL

>> + Install PostgreSQL `sudo apt-get install postgresql`

>> + Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`

>> + Login as user `"postgres" sudo su - postgres`.This is automatically created user by postgres.

>> + Get into postgreSQL shell `psql`

>> + Create a new database named catalog and create a new user named catalog in postgreSQL shell

>>>> + `postgres=# CREATE DATABASE catalog;`
>>>> + `postgres=# CREATE USER catalog;`
#### Set a password for user catalog
>> + postgres=# `ALTER ROLE catalog WITH PASSWORD 'password'`;
###### Give user "catalog" permission to "catalog" application database
>> + postgres=`# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog`;
>> + Quit postgreSQL `postgres=# \q`

## Install git, clone and setup your Catalog App project.
>> + Install Git using `sudo apt-get install git`
>> + `cd /var/www` to move to the /var/www directory
>> + Create the application directory `sudo mkdir catalog`
>> + Move inside this directory using `cd catalog`
>> + Clone the Catalog App to the virtual machine from [here](https://github.com/Abhi-sigma/catalog)
>> + Move to the inner catalog directory using `cd catalog`
>> + Edit database_setup.py `create_engine('sqlite:///catalog.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')`
>> + Install pip `sudo apt-get install python-pip`
>> + Install `sudo pip install virtualenv`
>> + Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
>> + Create database schema `sudo python database_setup.py`
>> + Install virtualenv: `sudo pip install virtualenv`
>> + Set enviornment name using : `sudo virtualenv venv`
>> + For automatic reuirements.txt pipreqs is used.Install by `pip install pipreqs`
>> + Generate the requirements.txt file by `pipreqs /path/to/project`
>> + Activate venv by invoking `souce/venv/bin/activate`
>> + Install all the requirements by using `pip install -r requirements.txt`
>>> + Run the following command to test if the installation is successful and the app is running: python main.py
It should display "Running on http://127.0.0.1:5000/". If you see this message, you have successfully configured the app.
if any import error `pip install` that module while venv is activated.
>> + To deactivate the environment : deactivate

##Install WSGI Interface
>> + WSGI (Web Server Gateway Interface) is an interface between web servers and web apps for python. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. So the first step is to install python-dev (mod-wsgi is already installed ) `sudo apt-get install python-dev`
>> + To enable mod_wsgi, run `sudo a2enmod wsgi`


## Configure Apache
>> + `sudo nano /etc/apache2/sites-available/catalog.conf`
>> + pasted the following,save and close
 ``` 
  <VirtualHost *:80>
    ServerName 13.210.20.118
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost> 
```
>>>> + The ServerName is the public ip address
>>>> + WSGIDaemonProcess is the process that starts when the request arrives and points to python virtual environment path
from where the modules are loaded
>>>> + WSGIScriptAlias is the path to catalog.wsgi which is a python file required for python  mod_wsgi interface
>> + Enable Virtual Host 'sudo a2ensite catalog'

## Create the wsgi file
>> + `cd into `/var/www/catalog`
>> + `touch catalog.wsgi`
>> + `nano catalog.wsgi`
>> + Copy  and pasted the following code,save and exit 
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog")
#print sys.path

from main import app as application

 ```
What this snippet of code basically does is,adds the catalog folder into the path
and imports flask app as application entry point
>> + `sudo service apache2 restart` restarts apache service

#####Sources:[Digital Ocean Example](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps),[Digital Ocean mod_wsgi ](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps)

### Other Changes
>> + The clients secrete json files cant be accessed with the initial setup of application.So
change the line which reads client json to full path.for eg.:-
``` 
open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']

```
>> + Update all your oauth origins.

### Logs
>> + While configuring the apache server,check logs found in  `/var/log/apache2` directory.
They can be helpful to debug.



Note:Big thanks to[kongling893](https://github.com/kongling893/Linux-Server-Configuration-UDACITY),[anumsh](https://github.com/anumsh/Linux-Server-Configuration) who have taken great effort to write such detailed setup which was really helpful,[iliketomatoes](https://github.com/iliketomatoes/linux_server_configuration) in setting up the server and writing my own detailed setup process.












