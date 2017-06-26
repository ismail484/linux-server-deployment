# linux-server-deployment[Mohamed Ismail]

# Description
 - it's  a baseline installation of a Linux server and prepare it to host my  web application (catalog-app). the project includes  security server  configurations against  a number of attack vectors, also includes installation  a database server configuration (Postgres), and deploy one of my existing web application (catalog-app) onto it.
 
 # Configurations
 
 
 
 
### Step1:Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  
  1.  block all incoming connections on all ports : sudo ufw default deny incoming.
  2.  allow outgoing connections on all ports using sudo ufw default allow outgoing.
  3.  allow incoming connection for SSH(port 2200) using sudo ufw allow 2200/tcp.
  4.  allow incoming connection for HTTP(port 80) using sudo ufw allow 80/tcp.
  5.  allow incoming connection for NTP(port 123) using sudo ufw allow 123/udp.
  6.  check the added rules : sudo ufw show added.
  7.  enable the firewall : sudo ufw enable.
  8.  check the firewall status : sudo ufw status.
 

### Step2: Install and configure Apache to serve a Python mod_wsgi application

 1. Install Apache web server:
 
  - `sudo apt-get install apache2` 
  
  
 2. Open a browser and open your public ip address, e.g. 52.28.98.229 ,It should say 'It works!' on the top of the page.
 3.Install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools:
   $ sudo apt-get install python-setuptools libapache2-mod-wsgi
 4. Restart the Apache server for mod_wsgi to load:
    $ sudo service apache2 restart
 5.Create an empty Apache config file with the hostname:
   $ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf
 6. Enable the new config file:
   $ sudo a2enconf fqdn
   
### Step2 Install git, clone and setup catalop-app project

 1. Install Git:
   $ sudo apt-get install git
 2. Set your name, e.g. for the commits:
   $ git config --global user.name "YOUR NAME"
 3.  Set up your email address to connect your commits to your account:
   $ git config --global user.email "YOUR EMAIL ADDRESS"
