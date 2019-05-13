## Basic setting up of the application deployment:
After setting up the linux server, upload your app (either with secure copy or with a git clone)

Then install python, 


```
sudo apt install python3-pip

```
install virtualenv

```
sudo apt install python3-venv

```

Next, create a virtual environment in your project's folder.


```
python3 -m venv PineappleApp_Main/venv

```

Then activate the virtual environment 
(if you're in the folder with the virtualenvironment, 
if not, change the path accordingly)
```
source venv/bin/activate
```

Next, once the venv is active, install all of your requirements (remember to have a requirements.txt file for your project)

```
pip install -r requirements.txt
```

## Installing Apache2 and WSGI
We'll be using Apache2 for the server and WSGI to connect/translate the server to the python framework 

```
sudo apt-get install apache2 
```

```
sudo apt-get install libapache2-mod-wsgi-py3
```

make sure you're in the tilda directory ( ~ )


```

cd ~

cd /etc/apache2/sites-available/

```

upon inspection of this directory (`ls`), you'll see two files:

```
000-default.conf default-ssl.conf 
```

For basic http, we'll just secure copy the default, rename it our app and edit it.

```
sudo cp 000-default.conf yourProjectsName.conf

```



Then let's edit the few configuration file with the basic nano editor

```
sudo nano yourProjectsName.conf
```

Okay, to unpack this, we're adding from Alias to the ending tag of VirtualHost. 
The Alias is for locating our static files (css, js, default img, whatever is static)

To run the static files, we need to grant access to them, this is the first Directory tag (if you have more types of files, aka media or audio, add these paths here too)

Next, we need to allow Apache2 to use our apps' WSGI file, (for Django apps). 

After that we have to specify paths to the WSGI files, the main project's files, and then also the virtual environment's files. 

* Note, you can call the variable name in WSGIDaemonProcess, whatever you'd like, but it has to be called the same variable name in WSGIProcessGroup


```

<VirtualHost *:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    #ServerName www.example.com

    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
    # error, crit, alert, emerg.
    # It is also possible to configure the loglevel for particular
    # modules, e.g.
    #LogLevel info ssl:warn

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf

  Alias /static /home/yourUserName/yourProjectsName/static
  <Directory /home/yourUserName/yourProjectsName/static>
    Require all granted
  </Directory>


  <Directory /home/yourUserName/yourProjectsName/yourProjectsName>
    <Files wsgi.py>
      Require all granted
    </Files>
  </Directory>

  WSGIScriptAlias / /home/yourUserName/yourProjectsName/yourProjectsName/wsgi.py
  WSGIDaemonProcess django_app python-path=/home/yourUserName/yourProjectsName python-home=/home/yourUserName/yourProjectsName/venv
  WSGIProcessGroup django_app

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

Next, we need to have Apache2 to have Linux privilages for accessing and running our app

Note:
* The ```:www-data``` is refering to Apache2

* ```chown``` stands for change owner
* ```chmod``` stands for Change mode, for instance read/write abilities

More info on [chmod](https://www.computerhope.com/unix/uchmod.htm) and [chown](https://www.computerhope.com/unix/uchown.htm)




Changing Apache's ownership over the database (if you use a different db, change accordingly)

```
		sudo chown :www-data yourApplicationsName/db.sqlite3
```

next, we want to change Apache's abilities over the project itself

```
		sudo chown :www-data yourApplicationsName/
```

Changing modification privilages for the database and for the app itself

```
		sudo chmod 664 yourApplicationsName/db.sqlite3
```

```
		sudo chmod 775 yourApplicationsName/
```





you can check this with

```
ls -la
```
and depending on what directory you are in, it'll tell you who has authorization on what. 


### Trouble Shooting with Apache2


If there is some syntax error in the file apache2.conf

In terminal:
```
    cd /etc/apache2
```
then :
```
    apache2ctl configtest
```
And it'll let you know what's happening (more than the default error when just running the app)




Here are some Apache2 functions

```

    # enable site
    sudo a2ensite
     
    # disable site
    sudo a2dissite
     
    # enable an apache2 module
    sudo a2enmod
     
    # e.g. a2enmod php4 will create the correct symlinks in mods-enabled to allow the module to be used. In this example it will link both php4.conf and php4.load for the user
     
    # disable an apache2 module
    sudo a2dismod
     
    # force reload the server:
    sudo  /etc/init.d/apache2 force-reload
    
```
