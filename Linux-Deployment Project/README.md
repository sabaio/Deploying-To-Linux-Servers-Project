#This is the final project for Udacity's Full Stack Web Developer Nanodegree.

###This page contains the Item-Catalog-Project built earlier in project 2 in the course.

###Here is the IP address 35.154.125.99 and SSH address is 2200.

###To vist Item-Catalog-Project here is the link http://35.154.125.99.xip.io.

##Creating an instance using AWS.
*After you login into the AWS site, in the services tab go to Amazon Lightsail, then press on create instance.
*Choose the nearest region to your location.
*Choose Linux/Unix platform, OS Only and Ubuntu 16.04 LTS.
*Choose an instance plan (I took $3.50 per month).
*Keep the default name provided by AWS (recommended) or rename your instance.
*Click the Create button to create the instance.
*Wait for the instance to start up.

##SSH into the server
*In Amazon Lightsail and from the account menu download the default private key by pressing the download button.

*Place the file LightsailDefaultPrivateKey-*.pem that contains the private key in /C:/Users/<username>/.ssh, rename the file with lightsail_key.rsa.
*Type in the terminal chmod 600 ~/.ssh/lightsail_key.rsa.

* Connect to your instance by typing in the terminal ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.154.125.99 (35.154.125.99 is my public IP address).


##Updating and upgrading the packages previously installed
* Update the packages using: sudo apt-get update.
* Upgrading the packages using: sudo apt-get upgrade.

##Changing the SSH port from 22 to 2200
*Edit the /etc/ssh/sshd_config file: sudo nano /etc/ssh/sshd_config.

*In line number 5 change the port number from 22 to 2200.

*To save the changes press CTRL+X then confirm by pressing Y.

*Type: sudo service ssh ,to restart restart the SSH.

##Configuration of the UFW (Uncomplicated Firewall)
###In order to allow only the incoming connections for the SSH port 2200, HTTP port 80, UDP port 123 , run the following commands to configure the Ubuntu firewall:
*sudo ufw status                  # The UFW is inactive.

*sudo ufw default deny incoming   # Deny incoming traffic.

*sudo ufw default allow outgoing  # Enable outgoing traffic.

*sudo ufw allow 2200/tcp          # Allow incoming tcp packets in port 2200.

*sudo ufw allow www               # Allow HTTP traffic.

*sudo ufw allow 123/udp           # Allow incoming udp packets in port 123.

*sudo ufw deny 22                 # Deny tcp and udp packets in port 53.

###In order to enable the UFW
run: sudo ufw enable. 
To make sure that is enabled you should have the following output: 
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
Firewall is active and enabled on system startup.

###In order to check the UFW status,
 run: sudo ufw status.
You should have the following output:
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
123/udp                    ALLOW       Anywhere                  
22                         DENY        Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)             
123/udp (v6)               ALLOW       Anywhere (v6)             
22 (v6)                    DENY        Anywhere (v6)

###Close the SSH connection by typing: exit.

###In the Amazon Lightsail instance, click on the three dots, then click on manage, then the networking tab.
<img src="img/three-dots.png">
Go to the firewall settings, delete SSH port 22 and add the TCP port 2200, and add the UDP port 123.
<img src="img/fw.png">
From your local terminal, run: ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@35.154.125.99, where 35.154.125.99 is the public IP address of my instance.

##Use Fail2Ban to block the attackers

Fail2Ban is an intrusion prevention software framework that protects computer servers from brute-force attacks.

*Install Fail2Ban: sudo apt-get install fail2ban.

*Install sendmail for email notice: sudo apt-get install sendmail iptables-persistent.

*Create a copy of a file: sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local.

*Change the settings in /etc/fail2ban/jail.local file:
set bantime = 3600
destemail = useremail@domain
action = %(action_mwl)s 

*Under [sshd] change port = ssh by port = 2200.

*Restart the service: sudo service fail2ban restart.


##Create User grader
*While logged in as ubuntu, add user: sudo adduser grader.

*Enter a password (twice) and fill out information for this new user.

##Give sudo permission to user grader
*Edits the sudoers file: sudo visudo.

*Search for the line that looks like this:

###root    ALL=(ALL:ALL) ALL
*Below this line, add a new line to give sudo privileges to grader user, it should be like this:

###root    ALL=(ALL:ALL) ALL 
###grader  ALL=(ALL:ALL) ALL

*Save and exit using CTRL+X and confirm with Y.


##Create an SSH key pair for grader using the ssh-keygen tool

On the local machine:
*Run ssh-keygen

*Enter file in which to save the key (I gave the name id_rsa) in the local directory ~/.ssh

*Enter in a passphrase twice. Two files will be generated ( ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub)

*Run cat ~/.ssh/id_rsa.pub and copy the contents of the file

*Log in to the grader's virtual machine

*On the grader's virtual machine:

*Create a new directory called ~/.ssh (mkdir .ssh)

*Run sudo nano ~/.ssh/authorized_keys and paste the content into this file, save and exit

*Give the permissions: chmod 700 .ssh and chmod 644 .ssh/authorized_keys

*Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no

*Restart SSH: sudo service ssh restart
On the local machine, run: ssh -i ~/.ssh/grader_key -p 2200 grader@35.154.125.99.

##Configure the local timezone to UTC
*While logged in as grader, configure the time zone: sudo dpkg-reconfigure tzdata. You should see something like that:

###Current default time zone: 'UTC'

###Local time is now:      Fri Jan 18 5:35:16 UTC 2019.

###Universal Time is now:  Fri Jan 18 5:35:16 UTC 2019.

##Install and configure Apache to serve a Python mod_wsgi application
*While logged in as grader, install Apache: sudo apt-get install apache2.

*Enter public IP of the Amazon Lightsail instance into browser. If Apache is working

*My project is built with Python 3. So, I need to install the Python 3 mod_wsgi package: sudo apt-get install libapache2-mod-wsgi-py3.

*Enable mod_wsgi using: sudo a2enmod wsgi.


##Step 11: Install and configure PostgreSQL
*While logged in as grader, install PostgreSQL: sudo apt-get install postgresql.

*PostgreSQL should not allow remote connections. In the /etc/postgresql/9.5/main/pg_hba.conf file.

*Switch to the postgres user: sudo su - postgres.

*Open PostgreSQL interactive terminal with psql.

*Create the catalog user with a password and give them the ability to create databases:

###postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
###postgres=# ALTER ROLE catalog CREATEDB;

*List the existing roles: \du. The output should be like this:

 
###                                  List of roles
### Role name |                         Attributes                         | Member of 
###-----------+------------------------------------------------------------+-----------
### catalog   | Create DB                                                  | {}
### postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

*Exit psql: \q.

*Switch back to the grader user: exit.

*Create a new Linux user called catalog: sudo adduser catalog. Enter password and fill out information.

*Give to catalog user the permission to sudo. Run: sudo visudo.

*Search for the lines that looks like this:

###root    ALL=(ALL:ALL) ALL
###grader  ALL=(ALL:ALL) ALL
*Below this line, add a new line to give sudo privileges to catalog user.

###root    ALL=(ALL:ALL) ALL
###grader  ALL=(ALL:ALL) ALL
###catalog  ALL=(ALL:ALL) ALL
*Save and exit using CTRL+X and confirm with Y.


*Run psql and then run \l to see that the new database has been created. The output should be like this:

###                                  List of databases
###   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
###-----------+----------+----------+-------------+-------------+-----------------------
### catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
### postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
### template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
### template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
###           |          |          |             |             | postgres=CTc/postgres
###(4 rows)

*Exit psql: \q.

*Switch back to the grader user: exit.

##Clone and setup the Item Catalog project from the GitHub repository
*While logged in as grader, create /var/www/catalog/ directory.

*Change to that directory and clone the catalog project:
sudo git clone https://github.com/sabaio/catalog.git.

*From the /var/www directory, change the ownership of the catalog directory to grader using: sudo chown -R grader:grader catalog/.

*Change to the /var/www/catalog/catalog directory.

*Rename the app.py file to __init__.py using: mv app.py __init__.py.

*In __init__.py, replace line 27:

*delete or comment app.run(host="0.0.0.0", port=5000, debug=True)
and write app.run()

*DB_Setup.py, replace line 9:

*delete or comment engine = create_engine("sqlite:///itemcatalog.db")

*engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')

##Login authentication using Google
*Go to Google Cloud Plateform.

*Click APIs & services on left menu.

*Click Credentials.

*Create an OAuth Client ID (under the Credentials tab), and add http://35.154.125.99.xip.io as authorized JavaScript origins.
Add http://35.154.125.99.xip.io/	
http://35.154.125.99.xip.io/login	
http://35.154.125.99.xip.io/gconnect as authorized redirect URIs.

*Download the corresponding JSON file, open it and copy the contents.

*Open /var/www/catalog/catalog/client_secret.json and paste the previous contents into the this file.


##The virtual environment and dependencies installation
*After loggin as grader, install the  pip using: sudo apt-get install python3-pip.

*In order to install the virtual environment use: sudo apt-get install python-virtualenv

*Go to the /var/www/catalog/catalog/ directory.

*Create the virtual environment using: sudo virtualenv -p python3 venv3.

*The ownership should be change to grader with: sudo chown -R grader:grader venv3/.

* Now activate the new environment using: . venv3/bin/activate.

*Install the following dependencies:

pip install httplib2
pip install requests
pip install --upgrade oauth2client
pip install sqlalchemy
pip install flask
sudo apt-get install libpq-dev
pip install psycopg2

*Run python3 __init__.py, you should see the following line: 

      
   * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit.

*Deactivate the virtual environment using: deactivate.

##Setting up the virtual host

*In order to use Python 3, add to  /etc/apache2/mods-enabled/wsgi.conf file the following:

<p>WSGIPythonPath directory|directory-1:directory-2:... </p>
WSGIPythonPath /var/www/catalog/catalog/venv3/lib/python3.5/site-packages
*Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:

<VirtualHost *:80>
    ServerName 13.59.39.163
  ServerAlias ec2-13-59-39-163.us-west-2.compute.amazonaws.com
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

* In order to enable virtual host type: sudo a2ensite catalog.
The following screen prompt is returned:

Enabling site catalog.
To activate the new configuration, you need to run:
  service apache2 reload

*Reload Apache: sudo service apache2 reload.


##Setting up Flask application

*In order to create /var/www/catalog/catalog.wsgi file add the following lines:

activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

<p>!/usr/bin/python</p>
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "..."
Restart Apache: sudo service apache2 restart.


##Disable the default Apache site

*Disable the default Apache site using: sudo a2dissite 000-default.conf. The following prompt will be returned:
     Site 000-default disabled.
     To activate the new configuration, you need to run:
     service apache2 reload

*Reload Apache using: sudo service apache2 reload.

##Launch ItemCatalog Application

*Change the ownership of the project directories using:     sudo chown -R www-data:www-data catalog/.

*Restart Apache again using: sudo service apache2 restart.

* Now open your browser to http://35.154.125.99.xip.io

##References
*ServerPilot, How to Create a Server on Amazon Lightsail:
https://serverpilot.io/docs/how-to-create-a-server-on-amazon-lightsail

*Official Ubuntu Documentation, UFW - Uncomplicated Firewall. https://help.ubuntu.com/community/UFW

*TechRepublic, How to install and use Uncomplicated Firewall in Ubuntu. https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/

*DigitalOcean, How To Protect SSH with Fail2Ban on Ubuntu 14.04. https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04

