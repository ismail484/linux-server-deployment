# linux-server-deployment[Mohamed Ismail]

# Description
 - it's  a baseline installation of a Linux server and prepare it to host my  web application (catalog-app). the project includes  security server  configurations against  a number of attack vectors, also includes installation  a database server configuration (Postgres), and deploy one of my existing web application (catalog-app) onto it.
 
 - Simply open [CatalogApp](http://52.28.98.229/), and login using your Facebook or Google+ account , and create your favourite Sports Catalog .
 # Configurations
 
### Step1: User Management: Create a new user (grader)and give user the permission to sudo
 - Source: [AWS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/managing-users.html)
 
 1. Create a new user:
  - `adduser grader`
 2. To check the User(grader) information :   
     - `sudo apt-get install finger`   
     -  `finger grader`
 3. Give new user the permission to sudo
   - `sudo visudosudo`
  4. add the below line of code after root ALL=(ALL:ALL) ALL 
   -  `grader ALL=(ALL:ALL) ALL `
  5. You can check the grader entry by below command: 
     `sudo cat /etc/sudoers`
  - you can switch from user to other : e.g,`sudo su - grader`
  
### Step-2: Follow the instructions provided to authenticated SSH from local server into Aws instance
  1. from local server create authentication key :
    - `ssh-keygen`
    - `cat .ssh/id_rsa.pub `
  2. from AWS instace on the new user (grader), get the same authentication key:
   - ```mkdir .ssh
        chmod 700 .ssh
        touch .ssh/authorized_keys
        chmod 600 .ssh/authorized_keys
        sudo vi .ssh/authorized_keys```
        paste the key from point (1) then save
   
### Step-3 : Change the SSH port from 22 to 2200

 1. Edit :` nano /etc/ssh/sshd_config`
 2. change port from 22 to 2200
 3. change PermitRootLogin without-password to PermitRootLogin no . it is disable root login.
 4. change PasswordAuthentication from no to yes.
 5. add AllowUsers grader at end of the file so that we will login through grader.
 6. restart the SSH service : `sudo service ssh restart`
 7. add custom role on AWS to allow external port 2200 .
 8. login `ssh -v grader@52.28.98.229 -p 2200`
 
### Step-4:Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  
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
     
 ### Step-5: Configure the local timezone to UTC

   Source: [Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

  1. Open the timezone selection dialog:
   -  `sudo dpkg-reconfigure tzdata`
  2. Then chose 'None of the above', then UTC.
   - *Setup the ntp daemon ntpd for regular and improving time sync:
   `sudo apt-get install ntp` 
 3. *Chose closer NTP time servers:
  - Open the NTP configuration file:`sudo vim /etc/ntp.conf`
Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.

### Step-6: Install and configure LAMP Apache to serve a Python mod_wsgi application
   - Source: [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
 1. Install Apache web server:
     - `sudo apt-get install apache2` 
 2. Open a browser and open your public ip address, e.g. http://52.28.98.229 ,It should say 'It works!' on the top of the page.
 3.Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
     -  `sudo apt-get install python-setuptools libapache2-mod-wsgi`
 4. Restart the Apache server for mod_wsgi to load:
     - sudo service apache2 restart
 5.Create an empty Apache config file with the hostname:
     - `echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf`
 6. Enable the new config file:
     -  `sudo a2enconf fqdn`
   
### Step-7 Install git, clone and setup catalop-app project
 - Source: [GitHub](https://help.github.com/articles/set-up-git/#platform-linux)
 1. Install Git:
     -  `sudo apt-get install git`
 2. Set your name, e.g. for the commits:
     -  `git config --global user.name "YOUR NAME"`
 3.  Set up your email address to connect your commits to your account:
     -  `git config --global user.email "YOUR EMAIL ADDRESS"`
   
 ### step-8: Setup for deploying a Flask Application on Linux instance Ubuntu 
  Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
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
     ```#!/usr/bin/python
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
        Source [virtualenv ](https://pythontips.com/2013/07/30/what-is-virtualenv/)
     -  `sudo virtualenv venv`
   10. Enable all permissions for the new virtual environment (no sudo should be used within):
     Source: [Stackoverflow](https://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip)
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
        Source [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
     -  `sudo nano /etc/apache2/sites-available/catalog.conf`
   17. Paste in the following lines of code and change names and addresses regarding your application:  
     ``` <VirtualHost *:80>
           ServerName 52.28.98.229   
           ServerAdmin grader@52.28.98.229
           ServerAlias ec2-52-28-98-229.eu-central-1.compute.amazonaws.com
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
     ```#!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")
  
        from catalog import app as application
        application.secret_key = 'Add your secret key' ```  
  
22. Restart Apache:
  - `sudo service apache2 restart`
  
  
### Step-9: Clone GitHub repository and make it web inaccessible

 1. Clone project 3 solution repository on GitHub:
  - `git clone https://github.com/ismail484/catalog-app` 
 2. Move all content of created FSND-P3_Music-Catalog-Web-App directory to /var/www/catalog/catalog/-directory and delete the leftover empty directory.
 3. Make the GitHub repository inaccessible:
  - Source:[Stackoverflow](https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
 4. Create and open .htaccess file:
  - `cd /var/www/catalog/ and $ sudo vim .htaccess`
 5. Paste in the following:
  - `RedirectMatch 404 /\.git`
  
### Step-10: Install needed modules & packages

1. Activate virtual environment:
  - `source venv/bin/activate`
2. Install httplib2 module in venv:
  - `pip install httplib2`
3. Install requests module in venv:
  - `pip install requests` 
4. Install oauth2client.client:
  - `sudo pip install --upgrade oauth2client` 
5. Install SQLAlchemy:
  -  ```pip install flask-sqlalchemy
        sudo pip install sqlalchemy```
6. Install the Python PostgreSQL adapter psycopg:
  -  ```sudo apt-get build-dep python-psycopg
      sudo apt-get install python-psycopg2```
      
### Step-11: Install and configure PostgreSQL DB
   Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps) 
1. Install PostgreSQL:
 - `sudo apt-get install postgresql postgresql-contrib` 
2. Check that no remote connections are allowed (default):
 - `sudo vim /etc/postgresql/9.3/main/pg_hba.conf` 
3. Open the database setup file:
 -  `sudo vim database_setup.py`
4. Change the line starting with "engine" to (fill in a password):
 - `python engine = create_engine('postgresql://catalog:PW-FOR-DB@localhost/catalog')`
5. Change the same line in application.py,lotsofmenus.py respectively
6. Rename application.py:
 -  `mv application.py __init__.py`
7. Create needed linux user for psql:
 - `sudo adduser catalog (choose a password)` 
8. Change to default user postgres:
 -  `sudo su - postgres`
9. Connect to the system:
 - `psql` 
10. Add postgre user with password:
   - Sources: [Trackets Blog](http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html) and [Super User](https://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql)  
11. Create user with LOGIN role and set a password:
 - `# CREATE USER catalog WITH PASSWORD 'PW-FOR-DB'; (# stands for the command prompt in psql)`
Allow the user to create database tables:
 - `ALTER USER catalog CREATEDB;`
12. *List current roles and their attributes:`# \du` 
13. Create database:
 - `# CREATE DATABASE catalog WITH OWNER catalog;`
14.Connect to the database catalog # `\c catalog`
15. Revoke all rights:
 - `# REVOKE ALL ON SCHEMA public FROM public;`
16. Grant only access to the catalog role:
 - `# GRANT ALL ON SCHEMA public TO catalog;`
17. Exit out of PostgreSQl and the postgres user:
 - `# \q, then $ exit`
19. Create postgreSQL database schema:
 - `python database_setup.py` 
 
### Step-12: Run application
 
 1. Restart Apache:
  - `sudo service apache2 restart`
 2. Open a browser and put in your public ip-address as url, e.g. 52.28.98.229 - if everything works, the application should come up
 3. *If getting an internal server error, check the Apache error files:
  - Source: [A2 Hosting](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
 4. View the last 20 lines in the error log: `sudo tail -20 /var/log/apache2/error.log` 
 5. If a file like 'g_client_secrets.json' couldn't been found:
  - Source: [Stackoverflow](https://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory)
  
 ### Step-13 : Get OAuth-Logins Working

 - Source: [Udacity Forum](https://discussions.udacity.com/t/oauth-provider-callback-uris/20460) and [Apache](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html)

 1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.28.98.229, its  ec2-52-28-98-229.eu-central-1.compute.amazonaws.com
 2. Open the Apache configuration files for the web app: `sudo vim /etc/apache2/sites-available/catalog.conf` 
 3. Paste in the following line below ServerAdmin:
   - `ServerAlias HOSTNAME, e.g. ec2-52-28-98-229.eu-central-1.compute.amazonaws.com`
 4. Enable the virtual host:
  - `sudo a2ensite catalog` 
 5. To get the Google+ authorization working:
   a.  Go to the project on the Developer Console: https://console.developers.google.com/project  
   b.  Navigate to APIs & auth > Credentials > Edit Settings  
   c.  add in Credentials Tab :"javascript_origins":"http://ec2-52-28-98-229.eu-central-1.compute.amazonaws.com","http://52.28.98.229"
  "redirect_uris":["http://ec2-52-28-98-229.eu-central-1.compute.amazonaws.com/oauth2callback"],  
   d. . add in your key_secrets file(client_secrets.json):"javascript_origins":"http://ec2-52-28-98-229.eu-central-1.compute.amazonaws.com",
  "redirect_uris":["http://ec2-52-28-98-229.eu-central-1.compute.amazonaws.com/oauth2callback","http://52.28.98.229"]  
  - add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to   Authorized redirect URIs, e.g. http://ec2-52-28-98-229.eu-central-1.compute.amazonaws.com/oauth2callback  
 6. To get the Facebook authorization working:  
    -  Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/ .
    -  Click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
    - in  Client OAuth Settings add:  
      a. Web Ip e.g,http://52.28.98.229/ ,http://52.28.98.229/login
      b. Web Ip authenticated e.g,http://52.28.98.229/_oauth/facebook
      c. add in your secret keys(fb_client_secrets.json):  "redirect_uri" :["http://52.28.98.229/login","http://52.28.98.229/","http://52.28.98.229/_oauth/facebook"]
     
   -   To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

 ### Step-14 :(optional)Install Monitor application Glances 
 
   Sources:[Glances](https://www.maketecheasier.com/glances-monitor-system-ubuntu/):
    
   - `sudo apt-get install python-pip build-essential python-dev`  
   -  `sudo pip install Glances`  
   -  `sudo pip install PySensors`  
 
