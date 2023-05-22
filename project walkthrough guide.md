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

### Entity relationship diagram
# [![Lesson 3: Creating a database diagram](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/TdXxN-4w3_s)
#### posts

|     Key      |      Name      |       Type       |
| ------------ | -------------- | ---------------- |
| ------------ | Title(unique)  |    Char[200]     |
|  ForeignKey  |     Author     |    user model    |
|              |  Created Date  |    DateTime      |
|              |  Updated Date  |    DateTime      |
|              |     Content    |    TextField     |
|              | Featured Image | Cloudinary Image |
|              |     Excerpt    |    TextField     |
| Many to Many |      Likes     |    User Model    |
|              |  Slug (unique) |     SlugField    |
|              |     Status     |     Integer      |

- Author field: 
    - one to many relationship, field must have a foreign key value as the author value wil need to appear in multiple plases/posts
- Slug: 
    - a label that can be used as part of a URL
- likes:
    - Many to Many field required as multiple users should be allowed to like multiple blog posts

___

### Create Database Models
# [![Lesson 4: Building the Database models](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/X7cdN0SQrX0)

in blog > models.py, add the following:
1. first, import "User" from django
    - from django.contrib.auth.models import User
2. import cloudinary to get featured images linked to database
    - from cloudinary.models import CloudinaryField
3. create a tuple variable with the name "STATUS" with the value of (0, "draft), (1, "Published")
    - STATUS = ((0, "Draft"), (1, "Published"))
4. begin creating the actual tables within python classes:
    - class Post(models.Model):
        -  __class imports the Model function from django's models library__ 
    -     title = models.CharField(max_length=200, unique=True)
        - __Makes a character field, single line with a max length of 200, specified as unique__
    -     slug = models.SlugField(max_length=200, unique=True)
        - __A slug is a string that can only include characters, numbers, dashes, and underscores. It is the part of a URL that identifies a particular page on a website, in a human-friendly form__
    -     author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="blog_posts")  # noqa
        - __has a one-to-many relationship, so contains a foreignkey that can be used on other tables to identify it__
        - __on_delete=models.CASCADE means that if an author is deleted, all related items oertaining to blog_posts wil be deleted__
    -     updated_on = models.DateTimeField(auto_now=True)
        - __datetime field that is defaulting to the current date and time__
    -     content = models.TextField()
        - __text field for content input__
    -     featured_image = CloudinaryField('image', default='placeholder')
        - __field that mkaes use of the importing of cloudinary, so we can import images from our cloudinary db into the project__
    -     excerpt = models.TextField(blank=True)
        - __a field for an excerpt of the full post text__
    -     created_on = models.DateTimeField(auto_now_add=True)
        - __same as updated on__
    -     status = models.IntegerField(choices=STATUS, default=0)
        - __integer field that picks from the choices given in the STATUS variable, values in in the STATUS tuple above determine the choices__
    -     likes = models.ManyToManyField(User, related_name='blog_likes', blank=True)
        - __Many to Many field, which links related tables with the name blog_data__
    - 
    -     class meta:
    -         ordering = ['-created_on']
        - __structures entries in descending order__
    - 
    -         def __str__(self):
    -             return self.title
        - __this method returns the string definition of an object, it has to target itself or it doesn't work__
    - 
    -         def number_of_likes(self):
    -             return self.likes.count()
        - __will count the number of likes stored on the likes entry__
5. repeat this process for comments and make a table that stores comments.
heres the diagram from the video, the code above should be sufficient to figure out how to create this table

|     Key      |      Name      |       Type       |    Extra Info     |
| ------------ | -------------- | ---------------- | ----------------- |
|  ForeignKey  |      post      |    Post model    | Cascade on delete |
|              |      name      |    CharField     |  Max Length 80    |
|              |      email     |    EmailField    |                   |
|              |      body      |    TextField     |                   |
|              |   created_on   |   DateTimeField  | auto_now_add True |
|              |     Excerpt    |   BooleanField   |   default False   |

Also, meta fields will need to be added for the comment itself and by who it was by.
This can be done using a python f string, see the Comment class model in models.py for example.

6. once all of this is built, migrate the models into the database:
    - python3 manage.py makemigrations
    - python3 manage.py migrate
    - if you need to double check the tables before you try to migrate them, you can do a dry run with:
        - python3 manage.py makemigrations --dry-run

