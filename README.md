# Linux Server Configuration Project
This project will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers

## Quick start

| Name | Value |
|---|---|
|IP Address|54.70.30.46|
|SSH Port|2200|
|Username|grader|
|URL of Application|[http://ec2-54-70-30-46.us-west-2.compute.amazonaws.com](http://ec2-54-70-30-46.us-west-2.compute.amazonaws.com)|


To connect to EC2 instance you need the udacity_key.rsa (supplied separately in the submit process):
```
ssh -i .ssh/udacity_key.rsa grader@54.70.30.46 -p 2200
```

## Summary of software and configuration
### Performing basic server configuration
#### 1. Launch your Virtual Machine with your Udacity account and log in
Launched Amazon EC2 instance using this [link](https://www.udacity.com/account#!/development_environment) from Udacity. Then I accessed the EC2 instance using SSH with the following command:
```
ssh -i .ssh/udacity_key.rsa root@54.70.30.46
```

#### 2. Create a new user named grader and grant this user sudo permissions
1. Created a new user named **grader** using the following command:
  `$ sudo adduser grader`
2. Provide sudo access by creating the following file
  `$ sudo nano /etc/sudoers.d/grader`
3. Add the following content
  ```
  grader ALL=(ALL) PASSWD:ALL
  ```

#### 3. Ensure users have a secure password
1. Installed *libpam-cracklib* to ensure secure passwords:
  `$ sudo apt-get install libpam-cracklib`
2. Update **/etc/pam.d/common-password** file:
  `$ sudo nano /etc/pam.d/common-password`
3. Add the following line
```
password requisite pam_cracklib.so try_first_pass retry=3 minlength=12 lcredit=1 ucredit=1 dcredit=1 ocredit=1 difok=4
```
* **try_first_pass**: sets the number of times users can attempt setting a good password before the passwd command aborts
* **minlen**: establishes a measure of complexity related to the password length
* **lcredit**: sets the minimum number of required lowercase letters
* **ucredit**: sets the minimum number of required uppercase letters
* **dcredit**: sets the minimum number of required digits
* **ocredit**: sets the minimum number of required other characters
* **difok**: sets the number of characters that must be different from those in the previous password

#### 4. Configure the local timezone to UTC
1. Changed EC2 instance time zone to UTC:
  `$ sudo dpkg-reconfigure tzdata`

#### 5. Adding Key Based login to new user **grader**
1. Changed from root user to grader user:
  `$ su - grader`
2. Create .ssh directoy
  `$ mkdir .ssh`
3. Added file **.ssh/authorized_keys** and copied ssh public key contents of udacity_key to **authorized_keys**:
  `$ nano .ssh/authorized_keys`
4. Restricted permissions to .ssh and authorized_keys:
  `$ chmod 700 .ssh`
  `$ chmod 644 .ssh/authorized_keys`


#### 6. Update SSH settings
1. Update sshd_config file:
  `$ sudo nano /etc/ssh/sshd_config`
2. To disable remote login for root user:
  ```
  Update PermitRootLogin without-password to PermitRootLogin no
  ```
3. To host SSH on non-default port 2200:
  ```
  Update PasswordAuthentication yes to PasswordAuthentication no
  ```
4. To host SSH on non-default port 2200:
  ```
  Update Port 22 to Port 2200
  ```
5. Restart SSH service
  `$ sudo service ssh restart`


#### 7. Configure the Uncomplicated Firewall (UFW)

1. Check current firewall status:
  `$ sudo ufw status`
2. Deny incoming traffic:
  `$ sudo ufw default deny incoming`
3. Allow outgoing traffic:
  `$ sudo ufw default allow outgoing`
4. Establish rules. For SSH (port 2200):
  `$ sudo ufw allow 2200/tcp`
5. Establish rules. For HTTP (port 80):
  `$ sudo ufw allow www`
6. Establish rules. For NTP (port 123):
  `$ sudo ufw allow ntp`
7. Enable UFW:
  `$ sudo ufw enable`


#### 8. Update all currently installed packages
1. Update the list of available packages and their versions:
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:
  `$ sudo sudo apt-get upgrade`

#### 9. Install and configure Apache to serve a Python mod_wsgi application

1. Install Apache web server:
  `$ sudo apt-get install apache2`
2. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart the Apache server for mod_wsgi to load:
  `$ sudo service apache2 restart`

### Install git, clone and setup your Catalog App project

#### 1. Install and configure git

1. Install Git:
  `$ sudo apt-get install git`

#### 2. Setup for deploying a Flask Application on Ubuntu VPS

1. Extend Python with additional packages that enable Apache to serve Flask applications:
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi
  `$ sudo a2enmod wsgi`
3. Create a Flask app:
    1. Move to the www directory:
      `$ cd /var/www`
    2. Setup a directory for the app, e.g. catalog:
        1. `$ sudo mkdir catalog`
        2. `$ cd catalog` and `$ sudo mkdir catalog`
        3. `$ cd catalog` and `$ sudo mkdir static templates`
        4. Create the file that will contain the flask application logic:
          `$ sudo nano __init__.py`
        5. Paste in the following code:
          ```python
            from flask import Flask
            app = Flask(__name__)
            @app.route("/")
            def hello():
              return "Hello from Flask framework!!"
            if __name__ == "__main__":
              app.run()
          ```
4. Install Flask
    1. Install pip installer:
      `$ sudo apt-get install python-pip`
    2. Install virtualenv:
      `$ sudo pip install virtualenv`
    3. Set virtual environment to name 'venv':
      `$ sudo virtualenv venv`
    4. Enable all permissions for the new virtual environment (no sudo should be used within):
      `$ sudo chmod -R 777 venv`
    5. Activate the virtual environment:
      `$ source venv/bin/activate`
    6. Install Flask inside the virtual environment:
      `$ pip install Flask`
    7. Run the app:
      `$ python __init__.py`
    8. Deactivate the environment:
      `$ deactivate`
5. Configure and Enable a New Virtual Host#
    1. Create a virtual host config file
      `$ sudo nano /etc/apache2/sites-available/catalog.conf`
    2. Paste in the following lines of code and change names and addresses regarding your application:
    ```
      <VirtualHost *:80>
          ServerName 54.70.30.46
          ServerAdmin admin@54.70.30.46
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
    3. Enable the virtual host:
      `$ sudo a2ensite catalog`
6. Create the .wsgi File and Restart Apache
  1. Create wsgi file:
    `$ cd /var/www/catalog` and `$ sudo nano catalog.wsgi`
  2. Paste in the following lines of code:
  ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'Add your secret key'
  ```
7. Restart Apache:
  `$ sudo service apache2 restart`

#### 3. - Make Git directory web inaccessible
1. Create and open .htaccess file:
  `$ cd /var/www/catalog/` and `$ sudo nano .htaccess`
2. Paste in the following:
  `RedirectMatch 404 /\.git`

#### 4. Install needed modules & packages
1. Activate virtual environment:
  `$ source venv/bin/activate`
2. Install httplib2 module in venv:
  `$ pip install httplib2`
3. Install requests module in venv:
  `$ pip install requests`
4. Install oauth2client.client:
  `$ sudo pip install --upgrade oauth2client`
5. Install SQLAlchemy:
  `$ sudo pip install sqlalchemy`
6. Install the Python PostgreSQL adapter psycopg:
  `$ sudo apt-get install python-psycopg2`

#### 5. Install and configure PostgreSQL

1. Install PostgreSQL:
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Check that no remote connections are allowed (default):
  `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`
3. Open the database setup file:
  `$ sudo nano database_setup.py`
4. Change the line starting with "engine" to (fill in a password):
  ```python engine = create_engine('postgresql://catalog:catalog@localhost/catalog')```
5. Change the same line in catalog.py & loadgrocery.py respectively
6. Rename catalog.py:
  `$ mv catalog.py __init__.py`
7. Create needed linux user for psql:
  `$ sudo adduser catalog` (choose a password)
8. Change to default user postgres:
  `$ sudo su - postgre`
9. Connect to the system:
  `$ psql`
10. Add postgre user with password:
    1. Create user with LOGIN role and set a password:
      `# CREATE USER catalog WITH PASSWORD 'catalog';` (# stands for the command prompt in psql)
    2. Allow the user to create database tables:
      `# ALTER USER catalog CREATEDB;`
    3. *List current roles and their attributes:
      `# \du`
11. Create database:
  `# CREATE DATABASE catalog WITH OWNER catalog;`
12. Connect to the database catalog
  `# \c catalog`
13. Revoke all rights:
  `# REVOKE ALL ON SCHEMA public FROM public;`
14. Grant only access to the catalog role:
  `# GRANT ALL ON SCHEMA public TO catalog;`
15. Exit out of PostgreSQl and the postgres user:
  `# \q`, then `$ exit`
16. Create postgreSQL database schema:
  $ sudo python database_setup.py
17. Load application data:
  $ sudo python loadgrocery.py

#### 6. OAuth-Logins Changes

1. Open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address
2. Open the Apache configuration files for the web app:
  `$ sudo nano /etc/apache2/sites-available/catalog.conf`
3. Paste in the following line below ServerAdmin:
  `ServerAlias ec2-54-70-30-46.us-west-2.compute.amazonaws.com`
4. Enable the virtual host:
  `$ sudo a2ensite catalog`
5. Change needed for Google+ authorization to work:
  1. Add Go to the project on the Developer Console: https://console.developers.google.com/project
  2. Navigate to APIs & auth > Credentials > Edit Settings
  3. add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-54-70-30-46.us-west-2.compute.amazonaws.com/oauth2callback
6. Changes needed to get Facebook authorization to work:
  1. Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
  2. Click on your App, go to Settings and fill in public IP-Address including prefixed http:// in the Site URL field

#### 7. - Run application
1. Restart Apache:
  `$ sudo service apache2 restart`
2. Open a browser and put in URL = [http://ec2-54-70-30-46.us-west-2.compute.amazonaws.com](http://ec2-54-70-30-46.us-west-2.compute.amazonaws.com)


## Third-Party Resources
* [How to enforce password complexity on Linux](http://www.computerworld.com/article/2726217/endpoint-protection/how-to-enforce-password-complexity-on-linux.html)
* [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
* [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)
* [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Flask Deploying - mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
* [A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
* [Flask by Example - Setting Up Postgres, SQLAlchemy, and Alembic](https://realpython.com/blog/python/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/)
* [Instructions for SSH access to the instance](https://www.udacity.com/account#!/development_environment)
* [How To Configure SSH Key-Based Authentication on a Linux Server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
* [Ubuntu Time Management](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)
* [Make .git directory web inaccessible](http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)
* [How to view Apache log files](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
* [OAuth Provider callback uris](http://discussions.udacity.com/t/oauth-provider-callback-uris/20460)
* [Name-based Virtual Host Support](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html)

