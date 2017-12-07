# UDACITY - Linux Server Configuration
**Project objective** - Take a baseline installation of a Linux server and prepare it to host web applications. Secure server from a number of attack vectors, install and configure a database server, and deploy one of existing web applications onto it.

**Public IP address** - 13.210.135.75

**SSH Port** - 2200

**URL** - http://ec2-13-210-135-75.ap-southeast-2.compute.amazonaws.com

## 1. Create an instance on Amazon
  i. [Register on amazon lightsail or login as existing user](https://amazonlightsail.com/)

  ii. Select Instance location

  iii. Select a platform => *Choose Linux/Unix*

  iv. Select a blueprint => *OS only*

  v. Select Operating System => *Ubuntu*

  vi. Choose instance plan => *$5 per month*

  vii. Name your instance => *Ubuntu-512MB-Sydney-Final*

  viii. Click Create Button
Note: this may take a while to configure the instance
## 2. login amazon server remotely as 'ubuntu' & update software
```
# Download SSH key pair and save it in the base folder as 'lightsail-key.pem'
# from command line cd to the folder where 'lightsail-key.pem' is located
  $ chmod 400 lightsail-key.pem
# login remotely as 'ubuntu
  $ ssh -i lightsail-key.pem ubuntu@13.210.135.75
# update package source list to keep track of latest software
  $ sudo apt-get update
# install/upgrade the latest packages
  $ sudo apt-get upgrade
# Remove unnecessary modules
  $ sudo apt-get autoremove
# install finger package
  $ sudo apt-get install finger
# set time to UTC (Select time zone to 'UTC' when prompted)
  $ sudo dpkg-reconfigure tzdata
```
## 3. Add 'grader' as sudo user
```
# create a new user 'grader'
  $ sudo adduser grader
# Select a 'password', other fields are optional to fill.
# Check the new user has been created by running
  $ finger grader
# to check which users have 'sudo' authority run following command
  $ sudo ls /etc/sudoers.d
# give 'grader' sudo authority
  $ sudo nano /etc/sudoers.d/grader
# add following line to the above file
  grader ALL=(ALL) NOPASSWD:ALL
# save and exit file
# recheck to see that 'grader now has sudo authority by running following command
  $ sudo ls /etc/sudoers.d
```
## 4. Remote login & disable password based authentication
```
# to ensure privacy generate key pair locally by running following command on terminal
  $ ssh-keygen
# save file as '/Users/<username>/.ssh/linuxCourse'
# copy public key to to clip board
  $ cat ~/.ssh/linuxCourse.pub
# from linux terminal cd to 'grader' home directory
  $ cd /home/grader
  $ sudo mkdir .ssh
  $ cd .ssh
  $ sudo touch .ssh/authorized_keys
  $ sudo nano .ssh/authorized_keys
# paste public key in '.ssh/authorized_keys'and save the file

# set file permissions
  $ chmod 700 .ssh
  $ chmod 644 .ssh/authorized_keys

# login as grader from command line
  $ ssh -i ~/.ssh/linuxCourse grader@13.210.135.75

# to disable password based login and force key based authentication
  $ sudo nano /etc/ssh/sshd_config
# Change Password Authentication from 'yes' to 'no'

# to change SSH port from 22 to 2200 edit '/etc/ssh/sshd_config' and change `port 22` to `port 2200`

# to change ownership of .ssh folder and internal files from root to grader
  $ sudo chown grader:grader .ssh
  $ sudo chown grader:grader .ssh/authorized_keys

# restart ssh to activate new configurations
  $ sudo service ssh restart
```

## 5. Configure ports & firewall
```
# this is to configure firewall to ensure only necessary ports are open while all other ports are inactive to ensure maximum protection from hacking attacks

# check firewall status which is currently 'inactive'
  $ sudo ufw status

# Block all incoming ports as default option
  $ sudo ufw default deny incoming

# Allow all outgoing ports as default option
  $ sudo ufw default allow outgoing

# configure firewall to support necessary ports for application to work
# to allow SSH configuration
  $ sudo ufw allow ssh
  $ sudo ufw allow 2200/tcp
  $ sudo ufw allow www
  $ sudo ufw allow ntp

# activate/enable the firewall
  $ sudo ufw enable
  $ sudo ufw status
```

## 6. Configure basic Flask App
### Install dependencies
  `$ sudo apt-get install apache2 libapache2-mod-wsgi python-dev python-flask`
### open following file
  `$ sudo nano /etc/apache2/sites-enabled/000-default.conf`
  add following at the end of file before </VirtualHost>
  `WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi`
### enable WSGI
  Web Server Gateway Interface (WSGI) is an interface between web servers and web apps for python. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. To enable WSGI run the following command,
  `$ sudo a2enmod wsgi`
### setup the necessary folder structure
  - Final file/folder structure should look like this.

  ```
  |--------www  
  |--------FlaskApp
  |----------------FlaskApp
  |-----------------------static
  |-----------------------templates
  |-----------------------__init__.py
  |----------------flaskapp.wsgi
  ```

  - Run following commands to achieve above objective
 ```
    $ cd /var/www
    $ sudo mkdir FlaskApp
    $ cd FlaskApp
    $ sudo mkdir FlaskApp
    $ cd FlaskApp
```
### setup __init__.py file
  `$ sudo nano __init__.py`
  Paste following text and save the file.
  ```
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, Flask App is now Running !!!"
if __name__ == "__main__":
    app.run()
```
### Configure and Enable a New Virtual Host
  `$ sudo nano /etc/apache2/sites-available/FlaskApp.conf`
  add following code and save the file
  ```
<VirtualHost *:80>
    ServerName mywebsite.com
    ServerAdmin narejo@gmail.com.com
    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    <Directory /var/www/FlaskApp/FlaskApp/>
    	Order allow,deny
    	Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/FlaskApp/static
    <Directory /var/www/FlaskApp/FlaskApp/static/>
    	Order allow,deny
    	Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
### setup .wsgi file
  `cd /var/www/FlaskApp`
  `sudo nano flaskapp.wsgi`
  paste following code to .wsgi file
  ```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super secret key'
```
### Restart apache
  `$ sudo service apache2 restart`
Visit app IP Address which must display 'Hello World!' message

## 7. Install and configure PostgreSQL
```
# install PostGreSQL
  $ sudo apt-get install postgresql
#login as super user 'postgres'
  $ sudo su - postgres
#launch PostGreSQL
  $ psql
# create a new database catalog and create a new user catalog
  postgres#= CREATE DATABASE catalog;
  postgres#= CREATE USER catalog;
# Alert: Don't forget to put semicolon at the end of psql commands commands
# set a password for user catalog
  postgres#= ALTER ROLE catalog WITH PASSWORD 'catalog';
# Permit 'catalog' user permission to 'catalog' DATABASE
  postgres#= GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
# QUIT PostGreSQL
  postgres#= \q
  $ exit
```
## 8. Install required dependencies
```
# install required dependencies
  $ sudo apt-get install git
  $ sudo apt-get install python-psycopg2
  $ sudo pip install oauth2client
  $ sudo pip install requests
  $ sudo pip install httplib2
  $ sudo pip install sqlalchemy
```
## 9. Clone catalog repository to Amazon server
```
# clone 'catalog' repo
  $ cd /var/www/FlaskApp
  $ sudo git clone https://github.com/arnarejo/udacity-catalog-app.git

# remove unnecessary files and directories
  $ cd udacity-catalog-app
  $ sudo rm catalogwithusers.db database_setup.pyc .DS_Store
  $ sudo rm -rf .git README.md
```
## 10. configure the app
1. modify 'database_setup.py', 'populate_db.py' and 'project.py' to update database route from `engine = create_engine('sqlite:///catalogwithusers.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
1. in the 'project.py' file change `app.run(host='0.0.0.0', port=5000)` to `app.run()`
1. Whereever necessary, change 'json' files path from relative to absolute by attaching `/var/www/FlaskApp/FlaskApp/` before the json files path.
1. change file name 'project.py' to '__init __.py'
  `sudo mv project.py __init__.py`
  `$ cd ..`
1. move 'udacity-catalog-app' folder files to 'FlaskApp' folder and then delete 'udacity-catalog-app' folder
  `$ sudo mv -v /var/www/FlaskApp/udacity-catalog-app/* /var/www/FlaskApp/FlaskApp`
  `$ sudo rm -rf /var/www/FlaskApp/udacity-catalog-app`
## 11. Updating Google and Facebook for authorized urls
```
# Open Google APIs console and add following URL to Catalog App credentials, in 'Authorized Javascript Origins'
    http://13.210.135.75

# Add following to 'Authorized redirect URIs'
    http://ec2-13-210-135-75.ap-southeast-2.compute.amazonaws.com/

# Open Facebook API console, 'settings' under 'facebook login' and add API URL to 'Site URL'
    http://13.210.135.75
    http://ec2-13-210-135-75.ap-southeast-2.compute.amazonaws.com/
```
## 12. Launch the app
1. switch to main FlaskApp folder `cd /var/www/FlaskApp/FlaskApp`
1. setup database `sudo python database_setup.py`
1. populate the database `sudo python populate_db.py`
## list of useful hints/command
1. to check error message from server run following command
    `$ sudo tail -f /var/log/apache2/error.log`
1. to check firewall status
    `$ sudo ufw status`
## Reference Materials
1. Udacity course (Configuring Linux Servers)[https://www.udacity.com/course/configuring-linux-web-servers--ud299]
1. (Udacity discussion forum)[https://discussions.udacity.com/]
1. (Udacity forum discussion on Facebook Login URLs)[https://discussions.udacity.com/t/add-aws-light-sail-url-in-facebook/459464]
1. (Udacity forum discussion on how to setup firewall)[https://discussions.udacity.com/t/how-do-we-set-up-aws-firewall/470462/2]
1. Cloud Academy (Amazon Lightsail: How to set up your first instance)[https://cloudacademy.com/blog/how-to-set-up-your-first-amazon-lightsail/]
1. Digital ocean (How to deploy a flask application ubuntu) guide[https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps]
1. (How To Configure the Apache Web Server on an Ubuntu or Debian VPS)[https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps]
1. Google Search
1. Stackoverflow
