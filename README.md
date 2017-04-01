# Linux Server Configuration

This project was made as a part of [Udacity's Full Stack Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)'s project 5 - Linux Server Configuration. The aim of this project was to host a web application on an Amazon Web Server and configure it to serve our web app and protect it against the basic attack vectors. 

The final website can be visited on this link : http://ec2-52-35-43-246.us-west-2.compute.amazonaws.com. The website hosted is the Project 3 of the nanodegree program - **_Item Catalog_** and is a Restaurant Menu Application.

Please follow the steps in this documentation to successfully configure the aws instance and host the website.

# Accessing the webserver

A user : **_grader_** is created with sudo priveleges. The user has been given key-pair to ssh into the server. The IP address of the server is : **52.35.43.246** and the user can ssh into the system using this command :  
```shell
ssh grader@52.35.43.246 -p 2200  
```
	
_Note : Here -p flag is to mention the port number for SSH_
After this command, the user is expected to type in the passphrase to log into the server.

# Installed Packages

| Package Name    |Descripton     | 
| ----------------|:-------------:| 
| **finger**      | Displays an easy to read information about a user|
| **apache2**     | HTTP Server |
| **libapache2-mod-wsgi** |	hosts Python applications on Apache2 server|
|**ntp**|	Synchronizes time over a network|
|**postgresql**|	Postgresql Database server|
|**git**|	Version control system tools|
|**python-setuptools**|	An easy-install package to facilitate installing Python| 
|**sqlalchemy**|	ORM and SQL tools for Python|
|**flask**|	Microframework for web applications|
|**python-psycopg2**|	PostgreSQL adapter for Python|
|**oauth2** |	Authorization framework for third-party login (Google and Facebook)|
|**fail2ban**|	Protection against suspicious site activity by IP banning|
|**Glances**|	Application monitor for host bugs|

# Summary of Configuration

1. Setup Virtual Machine and SSH into the server.
2. A new system user grader was created with permission to sudo.
3. All cuurently installed packages were updated and upgraded.
4. CRON tasks added to update and upgrade installed packages.
5. Changed SSH Port from 22 to 2200 and configure SSH access.
6. Configured UFWto only allow incoming connections for SSH(Port:2200), HTTP(Port:80) and NTP(Port:123).
7. Configured local Time Zone to UTC.
8. Installed and configure Apache to serve a Python mod_wsgi application.
9. Installed Git and Setup Environment for delopying Flask Application.
10. Install and configure PostgreSQL with default settings to not allow remote connection.
11. Created a new user catalog, added user to PostgreSQL databse with limited permissions to catalog application database.
12. Get OAUTH-LOGINS (Google+ and Facebook) working.
13. Installed and Configured Fail2ban intrusion protection that bans suspicious IPs.
14. Installed Glances to view full system status.

_Now we will walkthrough the process of configuring the amazon server._

# A. Setup Virtual Machine and SSH into the server

A new development environment is created by visiting [Udacity's development environment](https://www.udacity.com/account#!/development_environment)(_only available to Udacity's Nanodegree students_). Follow the steps on the page to download the private key and set up the root user.Take a note of your public IP address - that will be used to visit the final app too.

If everything works out well you should be able to log in to the server using following command :  
```
ssh -i ~/.ssh/udacity_key.rsa root@52.35.43.246
```

# B. Create a new user with sudo permissions

1. Create a new user _grader_ :   
```
root@ip-10-20-11-198:~$ sudo adduser grader
```
You can check if the user was created by using the _finger_ application.  

```
root@ip-10-20-11-198:~$ sudo apt-get install finger
root@ip-10-20-11-198:~$ finger grader
```
2. Give the new user sudo permission:

(a) - Open sudoer configuration using visudo command:

    root@ip-10-20-11-198:~# visudo
    
A nano editor will open up. scroll down to the line that reads:

    root ALL=(ALL:ALL) ALL
    
(b) - Below that line, add the new user: grader like so:

    root ALL=(ALL:ALL) ALL
    grader ALL=(ALL:ALL) ALL
    
Save changes by pressing `ctrl+x` and press `y` and then press Enter key.

(c ) - You can list all the users present root.

    root@ip-10-20-11-198:~# cut -d: -f1 /etc/passwd
    
# C. Update and Upgrade Currently Installed Packages

Update all available packages: This will provide a list of packages to be upgraded.
    
    root@ip-10-20-11-198:~# sudo apt-get update  
    
Upgrade packages to newer versions:   
    
    root@ip-10-20-11-198:~# sudo apt-get upgrade   
    
Add CRON script to manage update and upgrade installed packages.

(a) - Install unattended-upgrade packages:

    root@ip-10-20-11-198:~# sudo apt-get install unanttended-upgrades
    
(b) - Enable the unattended-upgrade packages:

    root@ip-10-20-11-198:~# sudo dpkg-reconfigure -plow unattended-upgrades
    
# D. Configure SSH 

1. Change the SSH configuration file:

(a)  Access the config file using nano editor:

    root@ip-10-20-11-198:~# nano /etc/ssh/sshd_config
    
In the nano editor:

(b) Change `Port 22` to `Port 2200`.

(c.)  Change `PermitRootLogin without-password` to `PermitRootLogin no`

(d)  Change PasswordAuthentication no to PasswordAuthentication yes.

(e)  At the end of the file, add `UseDNS` no and `AllowUsers grader`. This will allow grader SSH login access.

(f) Exit nano editor: ctrl+x, y then enter, and Restart SSH service for changes.  
```
root@ip-10-20-11-198:~# sudo service ssh restart  
```  

Now you can connect to grader from local Machine using :  
```
YOUR LOCAL MACHINE:~$ ssh grader@52.35.43.246 -p 2200
```
    
2. Create a SSH Key Pair:

(a)  Switch to your local machine by using the exit command:

```
root@ip-10-20-11-198:~# exit
```  

Now enter this command to generate a SSH key pair.

```
YOUR LOCAL MACHINE:~$ ssh-keygen
```  

Now you are asked to give a filename for the key pair. You can change the **id_rsa** to whatever filename you want. 

You will also be prompted to enter a passphrase which will prevent unauthroized access to the files. **_Keep this passphrase safe. This will be used to ssh as grader everytime._**

Now you will see that ssh-keygen has generated two files : file_name (private_key) and file_name.pub (public_key) file. The file file_name.pub will be placed on the server for authorization.

(b) Switch to the remote server as `grader` and create a directory called `.ssh`
```
    grader@ip-10-20-11-198:~$ mkdir .ssh
```

Create a new file within the `.ssh` directory called `authorized_keys`. A  file that will store the public keys.

    grader@ip-10-20-11-198:~$ touch .ssh/authorized_keys
    
(c.)  Switch back to your Local Machine, and copy the contents of your_file.pub:
```
    YOUR LOCAL MACHINE:~$ sudo cat ~/.ssh/ida_rsa.pub
```

(d)  Switch back to your Remote Server, edit ```authorized_keys``` file and paste the content of ```your_file.pub``` inside. Save file.
```
    grader@ip-10-20-11-198:~$ sudo nano .ssh/authorized_keys
```

(e)  Set specific file permission on SSH and ```authorized_keys``` directories:
```
    grader@ip-10-20-11-198:~$ chmod 700 .ssh
    grader@ip-10-20-11-198:~$ chmod 644 .ssh/authorized_keys
```

(f) SSHD Configuration:
```
    grader@ip-10-20-11-198:~$ sudo nano /etc/ssh/sshd_config
```

Here,change the `PasswordAuthentication yes` to `PasswordAuthentication no`

# E. Configure the UFW 

1. Check the UFW status :  
```
grader@ip-10-20-11-198:~$ sudo ufw status
```  
It should be **_inactive_** currently.

2. The standard method to configure the firewall is to disallow all incoming connections and then only allow whatever is necessary. Follow the following commands : 

```
grader@ip-10-20-11-198:~$ sudo ufw default deny incoming
grader@ip-10-20-11-198:~$ sudo ufw allow 2200/tcp
grader@ip-10-20-11-198:~$ sudo ufw allow 80/tcp
grader@ip-10-20-11-198:~$ sudo ufw allow 123/udp
```

3. You can now enable the firewall :  
```
grader@ip-10-20-11-198:~$ sudo ufw enable
```

# F. Configure local Time Zone to UTC

1. Open Timezone selection dialog:  
```
grader@ip-10-20-11-198:~$ sudo dpkg-reconfigure tzdata
```  
A selection menu will open up. Choose **None of the above**,and then choose **UTC**.

2. Setup ntp daemon to improve time sync:  
```
grader@ip-10-20-11-198:~$ sudo apt-get install ntp
```

# G. Install and Configure Apache to serve a Python mod_wsgi application

1. Install Apache web Server:   
```
grader@ip-10-20-11-198:~$ sudo apt-get install apache2
```
If apache was installed correctly you should see a default page when you will visit your IP address from develop environment page : `52.35.43.246`.

2. Install mod_wsgi, and python-setuptools helper package. This will serve Python apps from Apache:  
```
grader@ip-10-20-11-198:~$ sudo apt-get install python-setuptools libapache2-mod-wsgi
```

3. Configure Apache to handle requests using the WSGI module
```
grader@ip-10-20-11-198:~$ sudo nano cat/etc/apache2/sites-enabled/000-default.conf
```  
Add the following line: `WSGIScriptAlias / /var/www/html/catalog.wsgi` at the end of the `<VirtualHost *:80>` block, right before the closing `</VirtualHost>`. Now save and quit the nano editor. Restart Apache: `sudo apache2ctl restart`

# H. Install git and configure apache to serve Flask Application

1. Install Git:  
```
grader@ip-10-20-11-198:~$ sudo apt-get install git
```

2. Add aditional library to support apache to serve Flask application :  
```
grader@ip-10-20-11-198:~$ sudo apt-get install libapache2-mod-wsgi python-dev
```

3. Enable ```mod_wsgi``` if not enabled already :  
```
grader@ip-10-20-11-198:~$ sudo a2enmod wsgi
```

4. Navigate to the ```wwww``` directory and follow the steps :  
```
grader@ip-10-20-30-101:~$ cd /var/www
```  
Create a directory called `catalog` and withing that, make another directory called `catalog`. The `/var/www/catalog` will house our wsgi application which points to the server file, in my case it's name is : _finalproject.py_. Also make two directories `static` and `templates` inside : `/var/www/catalog/catalog` directory. Follow these steps : 

```
    grader@ip-10-20-11-198:/var/www$ sudo mkdir catalog
    grader@ip-10-20-11-198:/var/www$ cd catalog
    grader@ip-10-20-11-198:/var/www$ sudo mkdir catalog
    grader@ip-10-20-11-198:/var/www/catalog$ cd catalog
    grader@ip-10-20-11-198:/var/www/catalog/catalog$ sudo mkdir static templates
```

3. Now install dependancies for running a Flask app and that are used within your app :
```
sudo apt-get install python-pip
sudo pip install Flask
sudo pip install Flask-SQLAlchemy
sudo pip install Flask-SeaSurf
sudo pip install oauth2client
sudo apt-get install libpq-dev
sudo pip install psycopg2
```

4. Configure and Enable a New Virtual Host that will house our ```.wsgi``` file we are to create. Create a virtual host config file : **_catalog.conf_**  
```
grader@ip-10-20-11-198:/var/www/catalog/catalog$ sudo nano /etc/apache2/sites-available/catalog.conf
```  

And type the following code :

```
<VirtualHost *:80>
      ServerName 52.35.43.246
      ServerAdmin abhighosh_18@yahoo.com
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

5. Create the `catalog.wsgi` file. The contents of the file is to point to the right application logic file. This file will be created in `/var/www/catalog` directory. Follow the steps : 
```
grader@ip-10-20-11-198:/var/www/catalog/catalog$ cd /var/www/catalog
grader@ip-10-20-11-198:/var/www/catalog$ sudo nano catalog.wsgi
```

Paste the following code when the nano editor opens up :
```
#!/user/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/catalog/")

from finalproject import app as application
application.secret_key = 'your_super_secret_key'
```
##### Please note that here I have mentioned `finalproject` because that is the file which houses my application logic. You should edit it accordingly if your file name is `__init__.py` or anything else.    

 6. Enable the virtual host :  
```
grader@ip-10-20-11-198:/var/www/catalog$ sudo a2ensite catalog.wsgi
```
and restart the apache service   
```
grader@ip-10-20-11-198:/var/www/catalog$ sudo service apache2 restart 
```  

7. Clone your git repository :   
```
grader@ip-10-20-11-198:/var/www/catalog$ sudo git clone https://github.com/GITHUB_USERNAME/PROJECT_NAME.git
```  
 Now follow these commands to copy the content into `/var/www/catalog/catalog/` :
 
```
grader@ip-10-20-11-198:/var/www/catalog$ mv Restaurant-Catalog-App/* /var/Catalog/catalog
grader@ip-10-20-11-198:/var/www/catalog$ sudo rm -f Restaurant-Catalog-App
```

# I. Set up Postgresql

Now we will install and configure **Postgresql**. We will also create a user - **catalog** with previleges to create database only which will be password protected. Follow the steps :

```
sudo apt-get install postgresql
```

Change the default user to postgres by typing : ```sudo su postgres``` and then type in ```psql```

```
postgres=# CREATE USER catalog WITH PASSWORD 'catalog';
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog WITH OWNER catalog;
```

Revoke all rights on the database schema, and grant access to catalog only.
```
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
```

Exit Postgresql and postgres user:
```
postgres=# \q
postgres@ip-10-20-11-110~$ exit
```

Now we should run the `database_setup.py` to create the database and `lotsofmenus.py` to populate the database initially. Note that inside these files your create engine should point to the new databse now :   
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

The basic syntax of this statement is:   

```
postgresql://username:password@host:port/database
```

# J. Configure Oauth credentials for 3rd party authentication 

1. Now that we are on a server the paths to `client_secrets.json` and `fb_client_secrets.json` should be changed to their absolute paths : `/var/www/catalog/catalog/client_secrets.json` and `/var/www/catalog/catalog/fb_client_secrets.json`. 

Wherever you see the old path you must change to this new absolute path. Common places are in `gconnect()` and `fbconnect()` functions and the place where `CLIENT_ID` is mentioned!

In **_gconnect():_**

```
app_token = json.loads(
	open(r'/var/www/Catalog/catalog/g_client_secret.json', 'r').read())['web']['client_id']

CLIENT_ID = json.loads(
    open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']

oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
```

In **_fbconnect():_**

```
app_id = json.loads(open('fb_client_secrets.json', 'r').
            read())['web']['app_id']    

app_secret = json.loads(
        open('fb_client_secrets.json', 'r').read())['web']['app_secret']
```        

2. We will now make necessary changes in Google developer's console and facebook developer's console : 

- Go to http://console.developers.google.com , select your project and navigate to _Credentials_ present on the left side menu. Edit your **Authorize Javascript Origins** to include your site's URL which in my case is : 
_http://ec2-52-35-43-246.us-west-2.compute.amazonaws.com_ and under **Authorize redirect URIs add the following : _http://ec2-52-35-43-246.us-west-2.compute.amazonaws.com/oauth2callback_.

- Go to http://developers.facebook.com and under **Settings** present on the left side of your screen add the following to the **Site URL** : http://ec2-52-35-43-246.us-west-2.compute.amazonaws.com and click on _Save_ changes.


# K. Install fail2ban and glances 

Fail2Ban is a service that can protect a system that is being targeted by brute force attacks or other botting threats. We will configure Fail2Ban to protect our ssh connection, as well as Apache itself.

1. Install fail2ban

```
grader@ip-10-20-11-198:~$ sudo apt-get install fail2ban
```

2. Copy the default config file

```
grader@ip-10-20-11-198:~$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

3. Open jail.local and change the followinf default parameters:

```
grader@ip-10-20-30-198:~$ sudo nano /etc/fail2ban/jail.local
```

Set the following parameters:

```
set bantime = 1600
destemail = abhighosh_18@yahoo.com
action = %(action_mwl)s
under [ssh] change port = 2200
```
4. Stop and restart the service again :

```
grader@ip-10-20-11-198:~$ sudo service fail2ban stop
grader@ip-10-20-11-198:~$ sudo service fail2ban start
```

# L. Final Steps :

To see the app live visit : http://ec2-52-35-43-246.us-west-2.compute.amazonaws.com/. If you face any errors go to `/var/log/apache2/error.log` to see the log.

-----------------------------


### Bibliography:

1. https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
2. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
3. http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/
4. http://killtheyak.com/use-postgresql-with-django-flask/
5. https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
6. https://www.digitalocean.com/community/tutorials/how-to-protect-an-apache-server-with-fail2ban-on-ubuntu-14-04
7. https://github.com/AbigailMathews/FSND-P5
8. http://askubuntu.com/questions/17823/how-to-list-all-installed-packages
9. http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
10. http://www.cyberciti.biz/faq/howto-restart-ssh/
11. http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user
12. https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#code
13. https://github.com/elnobun/Linux-Server-Configuration-FSND-P5


### Improvements:

The app runs into a problem when trying to visit the menu items without logging in. Also there are some broken image links. After logging out the authorised access to delete the restaurant and menu items is granted. Needs pagination.
