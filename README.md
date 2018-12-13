# Udacity Linux Server Configuration Project

### Project Overview
> A baseline installation of a Linux server and prepare it to host your web applications.
secure the server from a number of attack vectors,
install and configure a database server, and deploy [Catalog](https://github.com/EngDiesel/Egyptian-Tourism-Catalog) web applications onto it.

### INFO
- Server's Public IP: [18.197.254.112](18.197.254.112)
- SSH Port: 2200

## **Update the Packages on the Server.**
```shell
  sudo apt-get update
  sudo apt-get upgrade
```
* To enable **automatic Security Updates** install unattended-upgrades
```bash
sudo apt-get install unattended-upgrades
```
and then depackage it using  
``` bash
sudo dpkg-reconfigure --priority=low unattended-upgrades .
```

## **Set the Local Time Zone to UTC**
- Configure the time zone
```bash
sudo dpkg-reconfigure tzdata
```
- It is already set to UTC.

## **Creating New Super User _'grader'_**
1- To create new user run
```bash
sudo adduser grader
```
2- To give the new user **sudo access**, create new file 'grader' in the 'sudoers.d' directory.
```bash
sudo nano /etc/sudoers.d/grader
```
and then add the following text
```plain
  grader ALL=(ALL:ALL) ALL
```

## **Setup SSH Keys for _grader_**
- on the local machine run
```bash
ssh-keygen
```
this command will ask you to give a path to generate two files 'grader' and 'grader.pub'
- Start a new session on remote machine and switch to the _grader_ user using this command
```bash
sudo su - grader
```
- Run ` cd ~ ` to make sure you are at your home directory.
- Make a new directory using `mkdir .ssh`.
- Create a new file `authorized_keys` in the `.ssh/` directory you just created.
- Edit the new file `sudo nano .ssh/authorized_keys` and paste the content of the `grader.pub` file you just created on your local machine.
- Exit and save the file.
- Change the mode of both of the `.ssh/` directory and `authorized_keys` file using
```bash
  sudo chmod 700 .ssh/
  sudo chmod 644 .ssh/authorized_keys
```

## **Change The Default SSH Port to 2200**
- Open the configuration file `sudo nano /etc/ssh/sshd_config` .
- Edit the file so now
  - `Port 2200` instead of `Port 22`.
  - `PasswordAuthentication no` .
  - `PermitRootLogin no` .
- Save and exit the file .
- Run `sudo service ssh restart` .

## **UFW Configuration**

- First Close all incoming ports using `sudo ufw default deny all` .
- Open all outgoing ports using `default allow outgoing` .
- Allow the ssh port you just changed `sudo ufw allow 2200/tcp` .
- Allow the http port using `sudo ufw allow www` .
- Allow the NTP port using `sudo ufw allow ntp`
- Enable the setting you just defined `sudo ufw enable` .

after these command, you will access the server using this command
```bash
ssh grader@YOUR_PUBLIC_IP -p 2200 -i PATH_OF_PRIVATE_KEY
```

## **Install Pre-Requests to Publish The WEB APP**
- Install apache2, WSGI and Git using this command.
```bash
sudo apt-get install apache2 libapache2-mod-wsgi-py3 git
```
- Installing and Configuring *Postgresql*
```bash
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
```

- Then create the Database and the schema.
```bash
CREATE USER catalog WITH PASSWORD 'password';
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
```
**Note:** In `model.py` and `views.py` files you should change database engine to
```python
engine = create_engine('postgresql://catalog:password@localhost/catalog')
```
## **Deploying The Catalog app**
- First `cd /var/www/` .
- Make a new Directory `mkdir catalog/` .
- Change the Owner `sudo chown grader:grader catalog/` .
- Clone the Project using this command
```bash
git clone https://github.com/EngDiesel/Egyptian-Tourism-Catalog.git catalog
```
- `cd catalog/` to move into the project .
- Create the app file uing `nano catalog.wsgi` .
- Then put this code into it
```python
import sys
sys.stdout = sys.stderr
activate_this = '/var/www/catalog/env/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
sys.path.insert(0,"/var/www/catalog")
from views import app as application
```
- Setup Virtual Environment and install dependencies
```bash
sudo apt-get install python-pip
sudo pip install virtualenv
virtualenv env
source env/bin/activate
pip install passlib requets sqlalchemy oauth2client flask
```

## **Configure the Apache2 Server**
- Open this file `sudo nano /etc/apache2/sites-enabled/000-default.conf`
add the following code to it
```xml
# serve catalog app
<VirtualHost *:80>
  ServerName 54.202.207.12
  ServerAlias ec2-54-202-207-12.us-west-2.compute.amazonaws.com
  ServerAdmin ali.mahmoud@engineer.com
  DocumentRoot /var/www/catalog
  WSGIDaemonProcess catalog user=grader group=grader
  WSGIScriptAlias / /var/www/catalog/catalog.wsgi

  <Directory /var/www/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Require all granted
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
- Restart the Apache2 Server
```bash
sudo service apache2 restart
```

- You are now good to go on the http://PUBLIC_IP/

## References
- [Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
- [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
- [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [Set Up Apache Virtual Hosts on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
- [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
- [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)
- [UFW](https://askubuntu.com/questions/54771/potential-ufw-and-fail2ban-conflicts)
- [Fix locale issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
- [Ask Ubuntu](https://askubuntu.com/)
- [Stack Overflow](https://stackoverflow.com/)
