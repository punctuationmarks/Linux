# Deploying a Flask app on a Linux Server.
	- If you need help with setting up an Ubuntu Server, check out [this](https://duckduckgo.com) walkthrough

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

## Setting up config variables

create a app_config.json file in the /etc/ folder

```
sudo touch /etc/app_config.json
```


 all of your environmental variables (from your local machine) like so:
	 SIDE NOTE: NOT ALL OF THESE HAVE TO MATCH UP, FOR INSTANCE, MAYBE YOU DON'T HAVE A RESET_EMAIL_PASSWORD
   
```
sudo touch /etc/app_config.json

sudo nano /etc/app_config.json

```

Then add this to that file:

```
{

        "SECRET_KEY" : "value",
        "SQLALCHEMY_DATABASE_URI" : "value",
        "EMAIL_USERNAME" : "value@gmail.com",
        "SPOTIFY_API_KEY" : "value",
        "EMAIL_PASSWORD" : "value"

}
```

*Add the config variables to your application, this is fairly straightfoward, just
follow the framework convention*




Install nginx
[nginx docs](https://nginx.org/en/docs/)
 
 
 ```

sudo apt install nginx 

```


 install gunicorn with pip (just make sure it's in your virtual environment
 
  ```

pip install gunicorn

```


 SIDE NOTE: nginx is going to handle the static files, gunicorn is going to handle the python code
 to do this we need to remove some of the defaults and add some rules

 removing nginx default settings 
 

  ```

sudo rm /etc/nginx/sites-enabled/default

```


 create a new file for your rules
 
 
  ```

sudo nano /etc/nginx/sites-enabled/pineapple_app
```
 


 *this is what you'll create in that file:*
 
   location /static {} means you're handing all of the static files 
   location /  <- means if you go to our ip address
   proxy_pass means you're fowarding all of the python files to be handled by gunicorn
   local host is our IP address (since we're running this on a local server in a different state)
   and port 8000 since that's where gunicorn typically runs python code
   the rest of the code is telling nginx to pass it onto gunicorn 


  ```

server {
        listen 80;
        server_name 12.34.56.78;

        location /static {
                alias /home/john/PeoplePrototype/PineappleApp/static;

        }
		
		location / {
				proxy_pass http://localhost:8000;
				include /etc/nginx/proxy_params;
				proxy_redirect off;
		}

}
```


 next you'll need to allow port 8000 through the firewall
 
 
  ```

 sudo ufw allow http/tcp
 ```


 
 and since we're done with testing, delete the port 5000
 
   ```

sudo ufw delete allow 5000
 ```


 make sure all of this is enabled 
 
  
 ```

sudo ufw enable
```


 restart your server 
 
  ```

sudo systemctl restart nginx 
```



 but we're not done just yet! we still don't have gunicorn set up yet 
 fairly simple set up
 number of workers formula == (2 x number of cores on the machine) + 1
 need to see how many cores you have on your linux machine?
nproc --all

 
gunicorn code to run the app (with the basic if __name__=="__main__"
 or app = create_app())

_this is just used to make sure it's running, not for deployment_  
     
```
gunicorn -w 3 run:app

```



 BUT THERE'S MORE!!!
 if you close your remote connection to the server, it'll actually crash it 
 what we need is something that will constantly monitor gunicorn and 
 start it and restart it if it crashes. We'll use `supervisor` to remedy this.
 
 once that is installed, we'll just need to set up a config file for supervisior 


 creating file "pineapple_app_prototype.conf"
 
  
 ```
 sudo apt install supervisor 
```


  
  Then create and edit a .conf file:
  
 ```
 sudo nano /etc/supervisor/conf.d/pineapple_app.conf 
```

 
 in that file create this:
   ```

[program:pineapple_app_prototype]
directory=/home/john/PineappleApp
command=/home/john/PineappleApp/venv/bin/gunicorn -w 3 run:app
user=john
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stderr_logfile=/var/log/PineappleApp/PineappleApp.err.log
stdout_logfile=/var/log/PineappleApp/PineappleApp.out.log
 ```




 next we need to create the PineappleApp.err.log file that we're referencing
 the -p means "create anyfolder that isn't already there along the way"
 
```
sudo mkdir -p /var/log/PineappleApp
sudo touch /var/log/PineappleApp/PineappleApp.err.log
sudo touch /var/log/PineappleApp/PineappleApp.out.log
```
 

 
```
sudo supervisorctl reload
```
 
Then boom! supervisior is up and running and you've got yourself an app





 ## Notes about Nginx
   - won't accept client uploads too big (around 2mb)
   to change this: 
   
```
  sudo nano /etc/nginx/nginx.conf
```
   
  
   and add under main settings (last thing is typically types_hash_max_size)
   increased it to 10megabytes
   
   ```
   client_max_body_size 10M; 
```

   
   make sure to restart nginx for settigs to be updated
   

```
  sudo systemctl restart nginx
```
