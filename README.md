# Linux Server Configuration
Udacity Full Stack Web Developer Nanodegree Project : Linux Server Configuration

## IP address and SSH port
* Public IP : ~54.172.67.62~
* SSH port : ~2200~
*To connect: ssh -i id_rsa grader@54.172.67.62 -p 2200

## Complete URL to hosted web application
~~ http://54.172.67.62/catalog/2/

## Project Overview
Take a baseline installation of a Linux server and prepare it to host a web applications. Secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

### How to complete this project?
This project is linked to the [Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299) course, which teaches you to secure and set up a Linux server. By the end of this project, you will have one of your web applications running live on a secure web server.

### 1. Get your server.
To complete this project, you'll need a Linux server instance. We recommend using [Amazon Lightsail](https://lightsail.aws.amazon.com/) for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications. 
1. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system. Choose a plan, give your instance a hostname and wait for ot to start up.
2. Log into it with SSH from your browser. When you SSH in, you'll be logged as the `ubuntu` user. An SSH window logged into the server instance. From here, it's just like any other Linux server.

### 2. Update all currently installed packages
- One of the most important and simplest ways to ensure your system is secure is to keep your software up to date with new releases

```
sudo apt-get update     # update available package lists
sudo apt-get upgrade    # upgrade installed packages
sudo apt-get autoremove # automatically remove packages that are no longer required
```
### 3. Create a New User `grader` and give `grader` sudo  access

1. Create a new user named grader `sudo adduser grader`  
2. To confirm addition of the new user that should be listed in the output.`sudo cat /etc/passwd`
3. Grant sudo access to grader:In `sudo visudo` add: grader ALL=(ALL:ALL) ALL right below user privilege specifications and save the file(ˆx)
4. Login as grader `sudo login grader`


### 4. Create an SSH key pair for grader using the ssh-keygen tool

1. Create a directory on local machine and browse to It. Generate keys `ssh-keygen`, enter file in which to save the key and give the name id_rsa
2. Copy the public key that is inside `cat id_rsa.pub` 

### 5. Add Public Key to Server

```
#On Ubunto, logged in as grader:
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys #paste the copied key and save
sudo chmod 700 .ssh           # change folder permission
sudo chmod 644 .ssh/authorized_keys
sudo service ssh restart

```

### 6. Change the SSH port from 22 to 2200.

1. In the lightsail instance, add a custom port tcp 2200
2. `sudo nano /etc/ssh/sshd_config` # change port 22 to 2200 and save It and change `PermitRootLogin without-password` line to `PermitRootLogin no`
3. restart ssh service `sudo service ssh restart`

### 7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
- Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server.
- When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. 

```
sudo ufw status                 # check ufw status 
sudo ufw default deny incoming   # Deny any incoming traffic
sudo ufw default allow outgoing  # Enable outgoing traffic
sudo ufw allow 2200/tcp         # allow SSH on port 2200
sudo ufw allow 80/tcp           # allow HTTP on port 80
sudo ufw allow 123/udp          # allow NTP on port 123
sudo ufw enable                 # enable firewall
sudo ufw status                 # check ufw status after updates 

```
### 8. Configure the local timezone to UTC and locales.

```
sudo dpkg-reconfigure tzdata #Configure the time zone
sudo dpkg-reconfigure locales #Configure locales
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
```
### 9. Connect to the instance on local machine throught command line:
```
ssh -i id_rsa grader@54.172.67.62 -p 2200
```

### 10. Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache `sudo apt-get install apache2`  
2. Go to http://54.172.67.62/, if Apache is working correctly, a Apache2 Ubuntu Default Page will show up
3. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`
4. Start wsgi mode `sudo a2enmod wsgi`
5. Restart Apache `sudo service apache2 restart` 

### 11. Install and configure PostgreSQL
```
sudo apt-get -qqy install postgresql python-psycopg2 # Install PostgreSQL
sudo -u postgres psql # basic server set up
CREATE USER catalog  WITH PASSWORD 'catalog'; # create user named grader.
ALTER USER catalog CREATEDB ; #enable catalog to create database
CREATE DATABASE catalog WITH OWNER catalog;
\list to visualize and confirm all DB
\c grader # connect to database
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
\q # quit PostgreSQL
```
Inside the Flask application, the database connection is now performed with:

```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog') #edit the files catalog_database_setup.py and housesaleitems.py in accordance.
```

### 12. Install git, clone and setup your Catalog App project.

1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir FlaskApp`
4. Move inside this directory using `cd FlaskApp`
sudo chown -R grader:grader /var/www/FlaskApp #Change the owner of the directory 
5. Clone the Catalog App to the virtual machine `git clone https://github.com/Rafaela314/Catalog-Project.git`
6. Rename the project's name `sudo mv ./Catalog-Project ./FlaskApp`
7. Rename `Catalog.py` to `__init__.py` using `sudo mv Catalog.py __init__.py`
8. Install pip `sudo apt-get install python-pip`
9. Use pip to install dependencies `sudo pip install -r requirements.txt`
10. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
11. Create database schema `sudo python catalog_database_setup.py`
12. `sudo python catalog_database_setup.py`

### 13. Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/000-default.conf`
2. Add the following lines of code to the file to configure the virtual host. 

	```
	<VirtualHost *:80>
		ServerName 54.172.67.62
		ServerAdmin rafa.pgcavalcante@@gmail.com
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/static
		<Directory /var/www/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog /var/www/FlaskApp/log/error.log
		LogLevel warn
		CustomLog /var/www/FlaskApp/log/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

### 14. Create the .wsgi File

1. Create the .wsgi File under /var/www/FlaskApp: 
	
	```
	cd /var/www/FlaskApp
	sudo nano flaskapp.wsgi 
	```
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/")
	from FlaskApp import app as application
	application.secret_key = 'rafaela'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `

## References:
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
https://docs.sqlalchemy.org/en/latest/core/engines.html
https://www.digitalocean.com/community/tutorials/sqlite-vs-mysql-vs-postgresql-a-comparison-of-relational-database-management-systems
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
https://www.digitalocean.com/community/tutorials/how-to-set-up-time-synchronization-on-ubuntu-16-04
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
https://github.com/iliketomatoes/linux_server_configuration
https://github.com/kongling893/Linux-Server-Configuration-UDACITY/blob/master/README.md
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps# Linux-Server-Configuration
