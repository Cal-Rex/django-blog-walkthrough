## when starting the project
___

## Create an empty project

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

add these values to the heroku app as config vars under settings
| Key          | Value                     |
| -----------: | :------------------------ |
| DATABASE_URL | ELEPHANTSQL DATABASE LINK |
| SECRET_KEY   | SECRET KEY FROM ENV.PY    |
| PORT         | 8000                      |

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

## Deploying to Heroku

# [![Lesson 1a: deploying the empty project](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/Qhypx3Z2Heg)

1. create a cloudinary account
2. copy environment variable API key given on dashboard
3. paste it into env.py as an environment variable:
    - os.environ["CLOUDINARY_URL"] = "CLOUDINARYAPIKEY"
4. add this new environment variable to the Heroku config vars
5. add a temporary config var to heroku:
    - DISABLE_COLLECTSTATIC : 1
        - to mitigate having no static files yet for the time being
        - is removed when coming to final deployment
6. 'cloudinary_storage' now needs to be added to INSTALLED_APPS in settings.py
    - this needs to be added ABOVE 'django.contrib.staticfiles'
7. add 'cloudinary' to INSTALLED_APPS in settings.py
    - this goes below 'django.contrib.staticfiles', but above our newest 'APPNAME'
8. set your static files up to be linked with cloudinary storage:
    - under STATIC_URL in settings.py, add:
        - STATICFILES_STORAGE = 'cloudinary_storage.storage.StaticHashedCloudinaryStorage'
            - tells app to use cloudinary as storage for static files
        - STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
            - makes our base directory at the top of the file static
        - STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
            - not actually needed for this project but it is good practice to set
9. set up media storage
    - under the STATIC code written above, add:
        - MEDIA_URL = '/media/'
        - DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
            - allows the storing of media and images on cloudinary
10. add the directory for the app's templates:
    - under BASE_DIR, add
        - TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')
    - in settings.py navigate to TEMPLATES and change the value of the 'DIRS' Key to:
        - [TEMPLATES_DIR],
11. Add allowed hosts for heroku deployment
    - add your heroku app link (the link used when deployed) to the value of ALLOWED_HOSTS in settings.py
        - ALLOWED_HOSTS = ['HEROKU/APP/LINK.com', 'localhost']
            - also add local host so project can still be run from github
12. create the following top-level folders in the project:
    - media
    - static
    - templates
13. create a Heroku Procfile
    - add a new file, call it Procfile
    - add gunicorn to Procfile
        - web: gunicorn appname.wsgi
            - "web" indicates that this app handles http traffic
            - "wsgi" stands for Web Services Gateway Interface server
14. Deploy to Heroku
    - add, commit, psh everything
    - got to heroku and connect github repo to the heroku project
    - deploy the branch

___

