# Heroku Deploy with Django (simple CRUD, Jinja2 templating,  no JS frontend framework)


## ProTip:
- While running and debugging your deployment,
```
heroku logs --tail
```
- Even the free tier might make you enter a credit card :/




## START
- pip install gunicorn
	- Which is a WSGI HTTP server (which is how Django handles their web servers)
- pip freeze > requirements.txt

- need have git, init, add, commit, .ignore, whole shebang

## Aside:
- we'll be using a postgresql database with heroku, it is the default database for Django apps


## In the settings.py

```
import django_heroku
```

- heroku doesn't allow `python manage.py collectstatic` to be sent as is (just like a linux deploy)
- so you'll need to specify where you want the static files/static root in the project-name/settings.py
```
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

- Move secret key, debug and any sensitive information to environment variables (syntax is same for all env vars)

```
SECRET_KEY = os.environ.get("SECRET_KEY")
# SECURITY WARNING: don't run with debug turned on in production!

# interesting work around, since environment variables only allow strings
# setting this to a if/else statement of if the environment variable is 'True'
# then it'll turn into a boolean of TRUE
DEBUG = (os.environ.get('DEBUG') == 'True')


# Set allowed hosts to the heroku url

ALLOWED_HOSTS = ['project-name.herokuapp.com']

```


- In CLI, set all the configurations (one by one)
```
heroku config:set SECRET_KEY='672sdfgsdfg7786dsf8gc8b4safdgsdfgdsfda'


```



- Automatically setting the whitenoise configs and set up the initial database
```
django_heroku.settings(locals())
```

## In the main directory
- create a Profile (no .extension type) in the main directory

```
web: gunicorn project-name.wsgi

```



### This needs to be moved up higher ###
## To help with CLI
- Install django-heroku which will help set up whitenoise and some other devops and backend helpers

`pip install django-heroku`



# Running python commands in heroku OR Performing tasks like migrating the apps for the database
- Migrating Models into the database
```
heroku run python manage.py migrate
```

- Need a super user?
```
heroku run python manage.py createsuperuser
```


- You can also bash into the heroku server
	- Heroku runs Dynos which are Linux machines, so run any debian command you want
	- (some do not work since this is not a fully functioning Linux Ubuntu Server,
	 for instance nano is not installed on these Dynos
```
heroku run bash
```



### Trouble Shooting tips:
## Database
- If heroku doesn't automatically choose a database for you (or you want to change it)
- Example for postgresql database on the free tier

```
heroky addons:create heroku-postgresql:hobby-dev

```

- To see the postgresql database

`heroku pg`
