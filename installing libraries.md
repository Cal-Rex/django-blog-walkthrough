## when starting the project
___

# [![Lesson 1: Creating an empty django project](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/MagEuw9Q4T4)
### install
The following 3 packages / libraries are the first things to install when creating the project
- django and green unicorn (unicorn)
    - for walkthrough: package used:
        - pip3 install 'django<4' gunicorn
    - modern ver:
        - pip3 install django gunicorn
- postgres, dj database and psycopg2
    - for walkthrough: package used:
        - pip3 install dj_database_url==0.5.0 psycopg2
    - modern ver:
        - pip3 install dj_database_url psycopg2
- Coudinary
    - pip3 install dj3-cloudinary-storage

### create requirements.txt
- pip3 freeze --local > requirements.txt

### create a Django project in the workspace
- django-admin startproject PROJECTNAME .

### create the app within the project
- python3 manage.py startapp APPNAME
- the APPNAME then needs to be listed in the INSTALLED_APPS list variable in PROJECTNAME/settings.py

### migrate new changes to the newly created database
- python3 manage.py migrate

### check it runs
- python3 manage.py runserver

___

## Set up Heroku

1. just create a new app for now

___

## set up ElephantSQL

1. create a new instance
2. copy the new database code

___

## create env.py
import ElephantSQL database instance

add following to env.py:
- import os
- os.environ["DATABASE_URL"] = "ELEPHANTSQL DATABASE LINK"
- os.environ["SECRET_KEY"] = "copy in a secret key or make one"

___

## link up env.py and database to project
add the following code under the imports at the top of settings.py
- import os
- import dj_database_url
- if os.path.isfile('env.py'):
    - import env

change the value of SECRET_KEY in settings.py to:
- SECRET_KEY = os.environ.get('SECRET_KEY')

comment out the existing DATABASES sqlite3 database variable and add the following code:
- DATABASES = {
    - 'default': dj_database_url.parse(os.environ.get("DATABASE_URL"))
- }

with everything connected, migrate it all into manage.py
- python3 manage.py migrate

you can check this worked by going to ElephantSQL, going to the db, clicking the browser tab on the left and check under table queries to see if tables have been populated

___

