### Linux Server Configration & prepare it to host your web applications

** [url for the website after the configration process ](http://ec2-52-29-121-11.eu-central-1.compute.amazonaws.com/ "url for the website after the configration process ")
**

** public_ip: 52.29.121.11 **

** ssh_port : 2200 **

## Connect Locally
- create your vpn instance  in my case i used lightsail (ubuntu)
  **they will give you file to connect ssh local in your machine with name :** `LightsailDefaultPrivateKey-eu-central-1.pem  `

- open your terminal and write : 
`sudo ssh -vvv -i ~<path to file>/LightsailDefaultPrivateKey-eu-central-1.pem       ubuntu@<public_ip>`

##### you are now connected to your vpn by ubuntu user locally

## Create new user (grader)
in your vpn terminal write
`sudo adduser grader`
`nano  /etc/sudoers.d/grader`
then wrtie <<
>>grader ALL=(ALL:ALL) ALL

the last step to add grader to sudo group

change to this user by just write
`su - grader `

## Now lets make it more secure

1- make new  ssh key : 
- write `ssh-keygen` in your local machine ,  then entered the file name , you will get 2 files (the_file_name) & (the_file_name.pub) 
- go to  the (the_file_name.pub) data by using nano or ant editor  and copy it 
- go to your vpn and write this in the terminal
```shell
su - grader
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
- past the .pub file in the authorized_keys file and exit it 
- change the file permissions by write 
```shell
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
- at  last write
`service ssh restart`


- in your machine now  you can ssh connect to your instance using line 
`ssh -i <the_file_name_in_.ssh>grader@<public_ip>
`
or by just 
` ssh -v grader@<public_ip>` and the os will look for the file in .ssh folder

## Change ssh default port and add UFW 
`nano sudo vim /etc/ssh/sshd_config` 
1- change to port 2200 or any availabe number
2- PermitRootLogin no
then exit
`sudo service ssh restart`
## enable UFW 
```shell
sudo ufw default deny incoming
sudo ufw default allow outcoming
sudo ufw allow 80/tcp
sudo ufw allow 2200/tcp
sudo ufw allow 123/udp
sudo ufw enable 
```
# Update & Upgrade & edit timezone
```shell
sudo apt-get update
sudo apt-get upgrade
```
`sudo dpkg-reconfigure tzdata` choose utc for example 
# install apache2 & python wsgi
```shell
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo service apache2 restart
```
- check your public_ip in the browser ( you will get apache2 default page)

# install postgres & create the database 
` sudo apt-get install postgresql`
` sudo su - postgres` change to postgres user 
`psql` go to the shell
- create the database(data) and user(amr) then give the user all the permission to it 
```sql
CREATE DATABASE data;
CREATE USER amr;
 ALTER ROLE amr WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE data TO amr;
```
## clone your site (flask app in this example)
`sudo apt-get install git `
`cd /var/www`
` git clone <your project > ` 

- in my case i will code : https://github.com/AmrAnwar/cataloge-flask-web-app-udacity-project-

the proejct file name will be (Website) and the main file will be __init__.py
- `sudo apt-get install python-pip ` 
in the __init__.py folder 
```shell
sudo pip install virtualenv 
virtualenv venv  
. venv/bin/active 
pip install -r requirments.txt
python __init__.py # if get run on 127.0.0.1 the server now work fine in the localhost
```

## create .config file and wsgi file 
sudo nano /etc/apache2/sites-available/Website.conf
then write :
```shell

<VirtualHost *:80>
        ServerName <your_server_dns_or_ip>
        ServerAdmin grader
        WSGIScriptAlias / /var/www/Website/website.wsgi
        <Directory /var/www/Website/Website/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/Website/Website/templates
        <Directory /var/www/Website/Website/templates/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
`sudo a2ensite Website` for enable it 
- add wsgi file 
in your Website folder write :
`sudo nano website.wsgi` then past this :

```python
#!/usr/bin/python
import sys
import logging
activate_this = 'var/www/Website/Website/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Website/")

from Website import app as application
application.secret_key = 'szvszvszvsbert4etbRGWYy$^#$'
```
## restart apache then check (your_server_dns_or_ip).
`sudo service restart apache2`
- if you get error check : `sudo tail -f /var/log/apache2/error.log`


