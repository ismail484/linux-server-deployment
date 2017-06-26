# linux-server-deployment[Mohamed Ismail]

# Description
 - it's  a baseline installation of a Linux server and prepare it to host my  web application (catalog-app). the project includes  security server  configurations against  a number of attack vectors, also includes installation  a database server configuration (Postgres), and deploy one of my existing web application (catalog-app) onto it.
 
 # Configurations
 
 
 
 
### Step1:Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  
  1.  block all incoming connections on all ports :
    - `sudo ufw default deny incoming`
  2.  allow outgoing connections on all ports :
    - `sudo ufw default allow outgoing`
  3.  allow incoming connection for SSH(port 2200) :
    -  `sudo ufw allow 2200/tcp`
  4.  allow incoming connection for HTTP(port 80) using sudo ufw allow 80/tcp.
    - `sudo ufw allow 80/tcp`
  5.  allow incoming connection for NTP(port 123) using .
    - `sudo ufw allow 123/udp`
  6.  check the added rules :
     - `sudo ufw show added`
  7.  enable the firewall :
     - `sudo ufw enable`
  8.  check the firewall status : 
     - `sudo ufw status`
 

### Step2: Install and configure Apache to serve a Python mod_wsgi application

 1. Install Apache web server:
   - `sudo apt-get install apache2` 
 2. Open a browser and open your public ip address, e.g. 52.28.98.229 ,It should say 'It works!' on the top of the page.
 3.Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
   -  `sudo apt-get install python-setuptools libapache2-mod-wsgi`
 4. Restart the Apache server for mod_wsgi to load:
   - sudo service apache2 restart
 5.Create an empty Apache config file with the hostname:
   - `echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf`
 6. Enable the new config file:
   -  `sudo a2enconf fqdn`
   
### Step3 Install git, clone and setup catalop-app project

 1. Install Git:
   -  `sudo apt-get install git`
 2. Set your name, e.g. for the commits:
   -  `git config --global user.name "YOUR NAME"`
 3.  Set up your email address to connect your commits to your account:
   -  `git config --global user.email "YOUR EMAIL ADDRESS"`
   
 ### step4: Setup for deploying a Flask Application on Linux instance Ubuntu 

  1. Extend Python with additional packages that enable Apache to serve Flask applications:  
    - `sudo apt-get install libapache2-mod-wsgi python-dev`  
  2. Enable mod_wsgi (if not already enabled):
    - `sudo a2enmod wsgi`
   Create a Flask app:
  3. Move to the www directory:
    - `cd /var/www`
  4. Setup a directory for the app, e.g. catalog:
    -  ```sudo mkdir catalog  
          cd catalog
          sudo mkdir catalog
          cd catalog
          sudo mkdir static templates ```  
  5.  Create the file that will contain the flask application logic:
   - `sudo nano __init__.py` 
  6. Paste in the following code:
    - ```#!/usr/bin/python
         from flask import Flask app = Flask(__name__)
         @app.route("/") def hello(): 
         return "Hello,world" 
         if __name__ == "__main__": 
         app.run() ``` 
   7. Install pip installer:
      - `sudo apt-get install python-pip`
   8. Install virtualenv:
      -  `sudo pip install virtualenv`
   9. Set virtual environment to name 'venv':
      -  `sudo virtualenv venv`
   10. Enable all permissions for the new virtual environment (no sudo should be used within):
     - `sudo chmod -R 777 venv` 
   11. Activate the virtual environment:
     -  `source venv/bin/activate`
   12. Install Flask inside the virtual environment:
     - `pip install Flask` 
   13. Run the app:
     - `python __init__.py` 
   14. Deactivate the environment:
      - `deactivate`
   15. Configure and Enable a New Virtual Host#
   16. Create a virtual host config file
      -  `sudo nano /etc/apache2/sites-available/catalog.conf`
   17. Paste in the following lines of code and change names and addresses regarding your application:  
     ``` <VirtualHost *:80>
           ServerName PUBLIC-IP-ADDRESS
           ServerAdmin admin@PUBLIC-IP-ADDRESS
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
         </VirtualHost>```
18. Enable the virtual host:
 -  `sudo a2ensite catalog`
19. Create the .wsgi File and Restart Apache
20. Create wsgi file:
  - `cd /var/www/catalog and $ sudo vim catalog.wsgi` 
21. Paste in the following lines of code:
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")
  
  from catalog import app as application
  application.secret_key = 'Add your secret key'
Restart Apache:
$ sudo service apache2 restart

