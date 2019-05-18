# Server Information
## IP Address
3.215.39.37
## SSH Port
2200
## Application URL
http://3.215.39.37.xip.io

# Summary of Software Installed
- apache2
- libapache2-mod-wsgi-py3
- python3-dev
- python3-pip
- virtualenv
- Flask
- httplib2
- oauth2client
- sqlalchemy
- sqlalchemy_utils
- psycopg2
- requests
- libpq-dev
- python3-dev
- postgresql
- postgresql-contrib

# Configuration Changes on Server
## Configuring Grader User SSH & Firewall
1. Attach static IP in
1. Configure Firewall
    1. Set ssh port to 2200
        - Edit ssh config file: ```sudo nano /etc/ssh/sshd_config  ```
        - Change row ```Port 22``` to Port ```2200```
        - Restart ssh service: ```sudo service sshd restart```
        - Edit server in online consol 
            - Add Custom row with Protocol:TCP Port:2200 to your Firewall settings in the Networking tab
            - Add Custom row with Protocol:UDP Port:123 to your Firewall settings in the Networking tab
            - Remove SSH line with Port:22
    1. Create firewall rules        
    ```sudo ufw status```
    ```sudo ufw default deny incoming```
    ```sudo ufw default allow outgoing```
    ```sudo ufw allow 2200/tcp```
    ```sudo ufw allow www```
    ```sudo ufw allow 123/udp```
    ```sudo ufw enable```
    ```sudo ufw status``` 


1. Add user (grader) with sudo rights
    1. Generate ssh keypair on local machine with ```ssh-keygen```
    1. Create new user on server
        1. Create user with: ```sudo adduser grader```
        1. You can see user added in /etc/passwd file
        ```sudo cat /etc/passwd```
        1. You can also see its home directory with ```ls /home```
        1. Create ssh directory to grader
        sudo mkdir /home/grader/.ssh
        1. Check .ssh direcotry created with ```ls -al /home/grader```
        1. Add authorized_keys file under grader's .ssh directory
        sudo touch /home/grader/.ssh/authorized_keys
        1. Edit the authorized_keys file, copy grader's public key to file
        ```sudo nano /home/grader/.ssh/authorized_keys```
        1. Change authorized_keys file permission to 600
        ```sudo chmod 600 /home/grader/.ssh/authorized_keys```
        1. Check new permission with ```ls -al /home/grader/.ssh```
        1. Change authorized_keys file ownership to grader:grader
        ```sudo chown grader:grader /home/grader/.ssh/authorized_keys```
        1. Check new ownership with ```ls -al /home/grader/.ssh```
        1. Change grader's .ssh folder permission to 700
        ```sudo chmod 700 /home/grader/.ssh```
        1. Check new permission with ```ls -al /home/grader```
        1. Change .ssh folder ownership to grader:grader
        ```sudo chown grader:grader /home/grader/.ssh```
        1. Check new ownership with ```ls -al /home/grader```
        1. Check your setup by signing in with new user
    1. Add sudo rights to grader
        1. Sing in with user who has sudo rights
        1. Create file grader in sudoers.d directory
        ```sudo touch /etc/sudoers.d/grader```
        1. Edit grader file under /etc/sudoers.d
        ```sudo nano /etc/sudoers.d/grader```
        Add the following code:
        ```grader ALL=(ALL) NOPASSWD:ALL```
        1. Check your configuration by executing a sudo command with user grader with command like
        ```sudo ls /etc/sudoers.d```
1. Disable remote login of root user and force SSH login
    - Edit sshd_config with: ```sudo nano /etc/ssh/sshd_config```.
    - Set ```PermitRootLogin no```.
    - Make sure config file has ```PasswordAuthentication no```.

## Update all system packages
```sudo nano apt-get update```
```sudo nano apt-get upgrade```

## Install apache2 and mod-wsgi
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3 python3-dev
sudo a2enmod wsgi
sudo service apache2 restart
```

## Clone project
```
cd /var/www
sudo mkdir catalog
sudo chown -R grader:grader catalog
cd catalog
git clone <project>
```

## Create .wsgi file
```
sudo nano /var/www/catalog/catalog.wsgi
```
Copy content:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
```

## Setup Virtual Environment
```
sudo apt-get install python3-pip
sudo pip3 install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
sudo pip3 install Flask
sudo pip3 install httplib2
sudo pip3 install oauth2client
sudo pip3 install sqlalchemy
sudo -H pip3 install psycopg2
sudo pip3 install sqlalchemy_utils
sudo pip3 install requests
```

## Create Virual Host for catalog app
Create Virtaul Host file
sudo nano /etc/apache2/sites-available/catalog.conf
Add content:
```
<VirtualHost *:80>
    ServerName 3.215.39.37
    ServerAlias 3.215.39.37.xip.io
    ServerAdmin ubuntu@3.215.39.37
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python3.5/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable catalog virtual host and disable the default one
```
sudo a2ensite catalog
sudo a2dissite 000-default.conf
```

## Create database

1. Install postgresql
```sudo apt-get install libpq-dev python3-dev```
```sudo apt-get install postgresql postgresql-contrib```

1. Create database in postgresql
- Change to user postgres 
```sudo -u postgres -i```
- Enter postgresql
```psql```
- In postgresql create catalog database and catalog user
```CREATE USER catalog WITH PASSWORD 'catalog';```
```ALTER USER catalog CREATEDB;```
```CREATE DATABASE catalog WITH OWNER catalog;```
- Connect to catalog database and configure
```\c catalog```
```REVOKE ALL ON SCHEMA public FROM public;```
```GRANT ALL ON SCHEMA public TO catalog;```
- Quit psql
```\q```
- Quit postgres user
```exit```

1. Setup Catalog database
- Go to project directory
```cd /var/www/catalog/catalog```
- Run ```python3 catalog_db_setup.py```
- Run ```python3 populate_catalog_db.py```

## Configure oauth2
- In OAuth consent screen added ```3.215.39.37.xip.io``` (it only takes, ```xip.io```)
- Credentials setting added to authorized redirect URIs:
```http://3.215.39.37.xip.io```
```http://3.215.39.37.xip.io/login```
```http://3.215.39.37.xip.io/gconnect```
- downloaded and replaced the new client_secret file
- in ```__init__.py``` edited ```flow_from_clientsecrets('/var/www/catalog/catalog/client_secret.json')``` to include path to client_secret file

## Other configuration changes
- Rename catalog.py to __init__.py
```mv catalog.py __init__.py```
- Changed user table to member table in database as user is a reserved word for postgresql

# Resources Used
- https://lightsail.aws.amazon.com/ls/docs/en_us/articles/lightsail-create-static-ip
- https://www.digitalocean.com/community/tutorials/ssh-essentials-working-with-ssh-servers-clients-and-keys
- https://lightsail.aws.amazon.com/ls/docs/en_us/articles/lightsail-how-to-set-up-putty-to-connect-using-ssh
- https://www.bogotobogo.com/python/Flask/Python_Flask_HelloWorld_App_with_Apache_WSGI_Ubuntu14.php
- http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
- http://postgresguide.com/utilities/psql.html

