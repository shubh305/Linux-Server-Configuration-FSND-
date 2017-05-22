# Linux Server Configuration Project



## Server Access Details


IP address: ```52.14.198.42```


Accessible SSH port: ```2200```


Application URL: [http://ec2-52-14-198-42.us-east-2.compute.amazonaws.com/](http://ec2-52-14-198-42.us-east-2.compute.amazonaws.com/)


## Instructions :


### 1. Create New User


  `useradd -m -s /bin/bash grader`


  Then open the sudoers.d file for the new user grader and give the sudo permission.


  `sudo nano /etc/sudoers.d/grader`


  Then add the following text in the editor:


  `grader ALL=(ALL) NOPASSWD:ALL`
 
 ### 2. Generate SSH login keys:
 
  `ssh-keygen`
 
  Create the .ssh directory and the authorized_key file for the user grader.
 
  `mkdir /home/grader/.ssh`
 
  `touch /home/grader/.shh/authorized_keys`


### 3. Give grader the owner permission and set the permissions of the .ssh and authorized_key file.


  `chown grader /home/grader/.ssh`


  `chown grader /home/grader/.ssh/authorized_keys`


  `chmod 700 /home/grader/.ssh`


  `chmod 600 /home/grader/.ssh/authorized_keys`


  Now add the public keys into the authorized_keys file.


  Open the sshd_config file.


  `sudo nano etc/ssh/sshd_config `

  #### Source : [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2)


### 4. Change the SSH port from 22 to 2200 and disable root login and password authentication.


  Find and set the following to this:-


  `Port 2200`

  `PermitRootLogin no`

  `PasswordAuthentication no`

  Save and close the file and restart ssh
  
  `sudo service ssh restart`

  Now login to the user 'grader' 


  `ssh -i [privateKeyFilename] grader@52.14.198.42`

### 5. Configure the UFW Firewall and enable it

  `sudo ufw default deny incoming`

  `sudo ufw default allow outgoing`

  `sudo ufw allow 2200/tcp`

  `sudo ufw allow www`

  `sudo ufw allow ntp`

  `sudo ufw enable`

  #### Source : [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-14-04)

### 6. Change timezone to UTC

  `sudo timedatectl set-timezone UTC`

### 7. Install Apache to serve a Python mod_wsgi application

  Install Apache:

  `sudo apt-get install apache2`

  Install the libapache2-mod-wsgi package for serving Python apps from Apache:

  `sudo apt-get install libapache2-mod-wsgi`

  Restart the Apache server for mod_wsgi to load:

  `sudo service apache2 restart`

  #### Source : [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

### 8. Install Git

  `sudo apt-get install git`

  Clone the gaming catalog app from github to the `/var/www/` directory.

  `cd /var/www`

  `sudo mkdir GamingCatalog`

  `cd /GamingCatalog`

  `git clone https://github.com/shubh305/Udacity-Item-Catalog-App-FSND.git`

  Rename the cloned directory to GamingCatalog

  `mv Udacity-Item-Catalog-App-FSND GamingCatalog`

  Create a catalog.wsgi file to serve the application over the mod_wsgi. That file should look like this:

  ```python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/GamingCatalog/")

  from GamingCatalog.catalog import app as application
  application.secret_key = 'super secret key'
```

### 9. Install project dependecies

  Move to the cloned GamingCatalog folder and do:

  `sudo pip-install -r requirements.txt`

### 10. Install and configure PostgreSQL

  Install postgresql:

  `sudo apt-get install postgresql postgresql-contrib`

  Check if any remote connections are not allowed.

  `sudo nano /etc/postgresql/9.1/main/pg_hba.conf`

  Create user and database catalog

  `sudo adduser catalog`
  `sudo passwd catalog`
  `sudo su postgres`
  `psql`
  `CREATE USER catalog; WITH PASSWORD '{password}'`
  `CREATE DATABASE catalog;`
  `\c catalog_app;`
  ` ALL ON SCHEMA public FROM public;`
  `GRANT ALL ON SCHEMA public TO catalog;`
  `\q`
  `exit`

  Change database from SQLite to PostreSQL in catalog.py, database_setup.py and lotsofcategories.py
  engine = create_engine('postgresql://catalog:{{password}}@localhost/catalog')

  #### Source : [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

### 11. Create and config new virtual host to serve the app.

  Create a virtual host conifg file: 

  `sudo nano /etc/apache2/sites-available/GamingCatalog.conf`

  Add the following lines to the nano editor:

  ```
  <VirtualHost *:80>
          ServerName 52.14.198.42
          ServerAdmin dazzling.shubh@gmail.com
          WSGIScriptAlias / /var/www/GamingCatalog/catalog.wsgi
          <Directory /var/www/GamingCatalog/GamingCatalog/>
                  Order allow,deny
                  Allow from all
          </Directory>
          Alias /static /var/www/GamingCatalog/GamingCatalog/static
          <Directory /var/www/GamingCatalog/GamingCatalog/static/>
                  Order allow,deny
                  Allow from all
          </Directory>
          ErrorLog ${APACHE_LOG_DIR}/error.log
          LogLevel warn
          CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```

  Disable the default virtual host with:

  `sudo a2dissite 000-default.conf`

  Then enable the catalog app virtual host:

  `sudo a2ensite GamingCatalog.conf`

  Reload Apache:

  `sudo service apache reload`

  The website will be live now.

### 12. Updating OAuth Google+ and Facebook

  For Google+:

  Go to Google Developers Console
  Add http://ec2-52-14-198-42.us-east-2.compute.amazonaws.com/goauth2redirect to the Authorized redirect URIs
  Facebook:

  Go to Facebook Developers Console
  Add http://ec2-52-14-198-42.us-east-2.compute.amazonaws.com to OAuth Redirect URI

### 13. Changing the path of credential/secret file in catalog.py

  Change the path of fb_client_secrets file to `/var/www/GamingCatalog/GamingCatalog/fb_client_secrets.json` whenever it occurs in the code (2 times).

  Change the redirect uri in fblogin and fboauth2callback from `localhost:500` to `http://ec2-52-14-198-42.us-east-2.compute.amazonaws.com/`

  Chanfe the path of g_client_secrets file to `var/www/GamingCatalog/GamingCatalog/g_client_secrets.json`


  Restart the Apache server

  `sudo service apache2 restart`

  #### Special thanks to [Steeve Wooding](https://github.com/SteveWooding) for taking his time and helping me out with the path   of secret files on zoom meeting in Udacity 1:1 appointment.

### 14. Install Glances (For extra credit)

  `sudo pip install Glances`

  To start Glances and monitor the application: 
  `sudo glances`

  #### Source : [Glances](https://nicolargo.github.io/glances/)
  To stop Glances press q

  ### Special thanks to [Steeve Wooding](https://github.com/SteveWooding) and [Sam Greenlee](https://github.com/sgreenlee) who wrote a very helpful readme.

  ### Special mention : [My First 5 Minutes On A Server; Or, Essential Security for Linux Servers](https://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-for-linux-servers)
