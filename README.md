
#  Udacity Full-Stack Web Developer Nanodegree Project: 
#  Linux Server Configuration

This is an [Amazon Lightsail](https://lightsail.aws.amazon.com) implementation of an Udacity Full-Stack Web Developer Nanodegree Project:Linux Server Configuration.

You can access the project at [http://52.79.156.71.xip.io/](http://52.79.156.71.xip.io/)


## Create new `grader` user and assign sudo permissions

*  ```sudo adduser grader```

* Add grader's sudo permissions into sudoer's file. Open  file `sudo visudo` and paste `grader ALL=(ALL:ALL) ALL` there right under `#User privilege specification`.

*  Add grader user to sudoers. Open  file `sudo nano /etc/sudoers.d/grader` and paste `grader ALL=(ALL:ALL) ALL` there.

*  Create `.ssh` directory for grader. `sudo mkdir /home/grader/.ssh` , copy content from `sudo vi /home/ubuntu/.ssh/authorized_keys` into `sudo vi /home/grader/.ssh/authorized_keys`

* Try to login via `ssh -i [keyFilename] grader@[ip-of-your-machine]"`


## Setting required firewall configurations

* Update port from `22` to `2200` in `/etc/ssh/sshd_config`.

* `sudo service ssh restart`

* Set following Amazon Lightsail:
1. Select your instance
2. Go to  `networking`
3. Set to following:
![ ](demo/demo.png)

* Check you have set that right via new ssh connection `ssh  -i [key]  grader@[lightsail-ip] -p 2200`

* Update configurations to defaults via `sudo ufw default deny incoming`  and `sudo ufw default allow outgoing`

* Allowing connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):
```
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp 
sudo ufw allow 123/udp
```
* `sudo ufw show added` to check that everything is set up right.

* Turn on the fw on `sudo ufw enable`.

* Check everything is set correct via `sudo ufw status`.


## Update the packages

```
sudo apt-get update
sudo apt-get upgrade
```

## Install Apache, PostgreSQL, Python and git

* Install Apache `sudo apt-get install apache2`.

* Install the libapache2-mod-wsgi package  `sudo apt-get install python-setuptools libapache2-mod-wsgi`

* Install PostgreSQL  `sudo apt-get install postgresql`

* Restart Apache `sudo service apache2 restart`

* Install git `sudo apt-get install git`

* Install python 2.7 `sudo apt-get install python-pip `


## Configure PostgreSQL


*  `sudo vim /etc/postgresql/9.5/main/pg_hba.conf` and  check if remote connections are prohibited

```
sudo su - postgres 
psql
postgres=# CREATE DATABASE sellingsocks;
postgres=# CREATE USER socks;
postgres=# ALTER ROLE socks WITH PASSWORD 'loveforsocks';
postgres=# GRANT ALL PRIVILEGES ON DATABASE sellingsocks TO socks;
\q
exit
```

## Setting up the project

* Preparing the project
```
mkdir /var/www/SellingSocks
cd /var/www/SellingSocks
sudo git clone -b postgre https://github.com/MadinaB/SellingSocks
cd SellingSocks
sudo mv application.py __init__.py
```
Update last line of  `__init__.py` from `app.run(host='0.0.0.0', port=8000) to app.run()`

* Run `sudo pip install -r requirements.txt`

* `sudo apt-get -qqy install postgresql python-psycopg2`

* `sudo python populate_socks_db.py`


## Running the project

* Create file with project's virtual host configurations `sudo nano /etc/apache2/sites-available/SellingSocks.conf`

* Copy this configuration into the file:
```
<VirtualHost *:80>
    ServerName SellingSocks
    ServerAdmin madinacodes@gmail.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    WSGIScriptAlias / /var/www/SellingSocks/sellingsocks.wsgi
    Alias /static /var/www/SellingSocks/SellingSocks/static
    <Directory /var/www/SellingSocks/>
        Order allow,deny
        Allow from all
    </Directory>
    <Directory /var/www/SellingSocks/SellingSocks/static/>
        Order allow,deny
        Allow from all
    </Directory>
</VirtualHost>
```

* Create the .wsgi file - `sudo vi /var/www/SellingSocks/sellingsocks.wsgi`
* Copy this code into the file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.append("/var/www/SellingSocks/")

from SellingSocks import app as application
application.secret_key = 'justsoooosecret'
```

Create static ip for your virtual machine and update host information on [google developers console](https://console.developers.google.com/apis/credentials/oauthclient/), then update client_secrets.json with new secrets.


Finally,

* `sudo apache2ctl restart`
* `sudo a2ensite SellingSocks`

You can see an error log in following file -`/var/log/apache2/error.log`




## Go to [see the socks](http://52.79.156.71.xip.io/)!



## Resources

[Udacity Full-Stack Web Developer Nanodegree](https://classroom.udacity.com/nanodegrees/nd004)
[Configuring Linux Web Servers](https://classroom.udacity.com/courses/ud299)
