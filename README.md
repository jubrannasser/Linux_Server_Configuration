# Linux Server Configuration



## About This Document
This document explain how to create instance  Linux server on [Amazon Lightsail](https://aws.amazon.com/lightsail/),  and perform the initial configuration of it. It explain how to  secure it and configure database server as well as running live a web application on web server.(my web app as  an example)

## My Server Info
- Domain Name: 18.184.158.74.xip.io 
- IP Address: 18.184.158.74
- SSh Port : 2200

##  Create Linux server Instance

 1. Open [Amazon Lightsail](https://aws.amazon.com/lightsail/)
 2. Create Account
 3. Once you're logged in, Lightsail will give you a friendly message with a robot on it,
    prompting you to create an instance. A Lightsail 
    instance is a Linux server running on a virtual machine inside an Amazon datacenter.
 5. Select OS Only. Second, choose any version of Ubuntu as the operating system.
 6. Choose your instance plan.
 7. Give your instance a hostname.
 8. Cilck Create, then Wait for it to start up.
 9. Once your instance is running, the display of
    server window gets brighter.
 10. click on ![IMg1](https://github.com/jubrannasser/Linux_Server_Configuration/blob/master/Capture.PNG) select manage, from the webpage that will open select networking tab then add (port 2200 )for ssh and (port 123 ) for NTP.(Port 80 is already present)
 11. Download the default private SSH key by navigating to the **Account Page** in the connect tab

 ##  Connect to server Instance
   1. at first time connect to server by browser ( from connect tab Click on _"Connect using SSH"_ ), in terminal:
       - run `nano /etc/ssh/sshd_config`
       - find line _port 22_
       - change 22  to 2200
       - find line _PermitRootLogin_
       - change option to _no_, to deny connect as root remotely
       - restart ssh service `sudo service ssh restart`
   2. close terminal, (After changing the port you will not be able to connect to the server via the browser)
      
   3. Place your downloaded private key in your local machine (linux,Mac or vagrant VM):
        - in terminal type `ssh ubuntu@[IP address] -p 2200 -i /path/of/private/key.pem`
   4. Now you are inside your instance  Linux server and ready to Configure it:
        - Update all application with the following commands:
          ```
          $ sudo apt-get update
          $ sudo apt-get upgrade
          $ sudo apt-get sudo apt-get autoremove
          ```  



## Linux Configuration

### User Management
 1. install finger (simple program to management users):
    ```
    $ sudo apt-get install finger
    ```
 2. add _grader_ user for udaciy grader:
    ```
     $ sudo adduser grader
    ```
    Then follow the prompt
 3. add _grader_ user to sudoers:
    - create grader file type `sudo nano /etc/sudoers.d/grader`
    - add following line to grader file:
      ```
      grader ALL=(ALL) NOPASSWD:ALL
      ```
 4. exit from remote server and on local machine generate key pairs:
    - run `ssh-keygen`
    - Locate path for pairs key
    - Enter your parspass for power security
    Now you have public and private key, go to path of public key file (with extent .pub) 
    and copy its contents
    
 5. Connect again to remote server `ssh ubuntu@[IP address] -p 2200 -i /path/of/ default private/key.pem`
    - type `su grader` to change user to grader
    - type `cd` to change work directory  to userhome directory
    - type `mkdir .ssh`
    - type `nano .ssh/authorized_keys` to create authoried_keys file
    - paste contents public key that copy it in '4' step and save file.
    - change permissions for .ssh directory `chmod 700 .ssh`
    - change permissions for authorized_keys `chmod 644 .ssh/authorized_keys`

 6. disable the password base logins (this default in Amazon Lightsail server so ignore):
    - type `sudo nano /etc/ssh/sshd_config
    - find _PasswordAuthentication yes_ line
    - change yes to no
 7. restart ssh server `$ sudo service ssh restart`
    
 8. Now we can login to remote server as grader user from local machine by command:
     ```
     $ssh grader@[ip address] -p 2200 -i /path/of/new/private/key
     ```

### Configure Firewall
we need to configure firewall to close all port that is no need to secure our server:
 ```
 $ sudo ufw default deny incoming
 $ sudo ufw default allow outgoing
 $ sudo ufw deny 22
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow 80/tcp
 $ sudo ufw allow 123/tcp
 $ sudo ufw enable
 ```
### Configure Database
 1. install postgresql `sudo apt-get install postgresql`
 2. create new user _catalog_ in postgresql:
 ```
 $ sudo -u postgres psql
 postgres=# CREATE USER catalog WITH PASSWORD '<password>';
 postgres=# ALTER USER catalog WITH CREATEDB;
 ```
 3. create database _catalogdb_:
 ```
 postgres=# CREATE DATABASE catalogdb WITH OWNER catalog;
 postgres=# /q  
 ```

### Setup Process for running web application
 Hosting this application will require the installing Dependencies:
```

$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install git
$ sudo apt-get install python python-pip
$ sudo -H pip install flask flask_httpauth
$ sudo -H pip install oauth2client
$ sudo -H pip install passlib
$ sudo -H pip install sqlalchemy 
$ sudo -H pip install requests
$ sudo -H pip install psycopg2-binary
```
### Configure apache2 Server
 1. type `sudo a2enmod wsgi` to enable WSGI
 2. cd to /var/www and create _Catalog_ directory
 3. cd to _Catalog_ directory and inside create  another _Catalog_ directory
 4. cd to /var/www/Catalog and type `sudo nano catalog.wsgi` to create catalog.wsgi file
 5. copy the following lines to it then save:
 ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/Catalog/")

  from Catalog import app as application
  application.secret_key = 'your_secret_key’
 ```
 6. type `nano /etc/apache2/sites-available/Catalog.conf` 
    to create _Catalog.conf_ file and copy the following line to it then save:
 ```
  <VirtualHost *:80>
        ServerName [ip address]
        ServerAdmin youre@email.com
        WSGIScriptAlias / /var/www/Catalog/catalog.wsgi

        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
 ```  
 7. `$ sudo a2ensite Catalog` to enable Catalog.conf
 8. `$ sudo a2dessite 000-default.conf` to disable 000-default.conf
 9. give permesion to www-data:
 ```
 $ sudo groupadd varwww
 $ sudo adduser www-data varwww
 $ sudo chgrp -R varwww /var/www
 $ sudo chmod -R 775 /var/www
 ```
 10. `service apache2 restart` to restart server and active changes

### Clone Item Catalog Project & Make interest changes
1. Clone the item catalog
```
$ cd /var/www/Catalog/Catalog
$ git clone [Repository URL] .
```
2. change name of "catalogapp.py" file to "__init__.py" `sudo mv catalogapp.py __init__.py`
3. `sudo nano __init__.py` and change:
   - database's Connecting string to 'postgresql://catalog:<password>@localhost/catalogdb'
   - UPLOAD_FOLDER path to '/var/www/Catalog/Catalog/static/img'
   - client_secrets.json file path to '/var/www/Catalog/Catalog/client_secrets.json'
4. `sudo nano db_setup.py` and change:
   - database's Connecting string to 'postgresql://catalog:<password>@localhost/catalogdb'
5. `sudo nano FillItem.py` and change:
   - database's Connecting string to 'postgresql://catalog:<password>@localhost/catalogdb'
6. `service apache2 restart` to restart server


### Google Sign-in 
We use Google API third-party to register users so we need to make interest changes to our app:
1. from Google API wbsite Update redirect url and javascript origins url that match our domain
2. download updated client_secrets.json file and copy content
3. in /var/www/Catalog/Catalog create _client_secrets.json_ file and paste content
(from step2) to it
4. open login.html file `sudo nano /var/www/Catalog/Catalog/templates/login.html` 
then change _'ClientId'_ to your client id
5. restart apache2 server

## References:
 - Udacity class room
 - https://github.com/Rawaneeta1/linux-server-configuration
 - [How to run pyhton flask on apache web server](https://www.youtube.com/watch?v=iVGtJOC71Fw)
 - [How To Use Roles and Manage Grant Permissions in PostgreSQ](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)
