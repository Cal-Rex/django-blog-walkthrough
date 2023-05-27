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

___

## building the admin and admin panel, and customising content input field on blog posts
# [![Lesson 5: Creating the admin panel](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/er5IKknKxoQ)

admin : 
| -------- | ----- |
| Username | admin |
| Password | admin |

1. we create an admin user using the terminal:
    - python3 manage.py createsuperuser
    - the terminal will then prompt you for username, email address and pw
        - pw wont be echod back to screen so you want be able to see what you're typing
2. admin page can be accessed by adding "/admin" to the end of the workspace url
3. add the Post model created in models.py to the Admin panel
    - in admin.py import the "Post" class from ".models"
        - from .models import Post
    - add the following code:
        - admin.site.register(Post)
4. add a WYSIWYG editor for the post
    - intall the summernote library
        - pip3 install django-summernote
        - documentation for summernote: https://summernote.org/
    - update the requirements.txt as we have installed a new library
        - pip3 freeze --local > requirements.txt
    - add summernote to list of INSTALLED_APPS in settings.py
        - use the syntax: django_summernote
    - the URLs now need to be set up, navigate to urls.py
        - on the import line for django.urls, add "import" to the list of libraries being imported
            - line should now look like this:
                - from django.urls import path, include
        - in the urlpatterns[] variable, add a new path for summernote:
            - path('summernote/', include('django_summernote.urls')), 
                - the include library that was just imported is used here
    - the admin panel now needs to be told what we want to use the summernote functionality for, which i this case, its to be used for the content field instead of a basic TextField
        - at the top of admin.py, import SummerNoteModelAdmin from django_summernote.admin
            - from django_summernote.admin import SummerNoteModelAdmin
            - with the new import, create a class called PostAdmin that inherits from SummerNoteModelAdmin, under the line:
                - admin.site.register(Post)
            - inside the class add the following:
                - summernote_fields = ('content')
                    - this links the summernote functionalt to fields with the name 'content'
            - above the class replace the following codeline:
                - admin.site.register(Post)
                    - we are replacing this because this method only allows for 2 arguments/parameters, so it fills up too quickly / has reduced functionality
            - with:
                - @admin.register(Post)
                    - this class decorator will register both the Post model and the Post admin class with the admin site.
        - migrate all these changes
            - because we havent made any structural database changes, the makemigrations command doesnt need to be run, just the migrate command.
                - python3 manage.py migrate
        - run the project
            - python3 manage.py runserver
        - then add /admin to the url once loaded in preview (the main site has no page so will come up with a debug screen)
        - if you try to add a new post you can see that summernote has overridden the content box with a text editor
        - make a pilot test post

___

## creating the admin panel, part 2
# [![Lesson 6: Creating the admin panel](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/JGt1p8JhPyY)

When we enter the post title, we want the  slug field to be generated automatically.
there is a variable in django that uses javascript to populate fields with prepopulated data.

in admin.py under the PostAdmin(SummernoteModelAdmin) class, add:
- prepopulated_fields = {'slug': ('title',)}
    - **the variable contains an object with a KVP, the key being the 'slug' field, and the value will be 'title' wrapped inside a tuple (as the value could contain more than one parameter)**

then, we can add a post filter function on the admin panel bu adding another line of code to the class:
- list_filter = ('status', 'created_on')

we can then add other functions from the django documentation that expand the details given of posts on the admin panel:
- list_display = ('title', 'slug', 'status', 'created_on')
    - **The list display method from django allows for table elements to be listed in a tuple as its parameter, then on the admin panel, these details will be displayed top level for the class/table**
    - documentation: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display
- search_fields = ['title', 'content']
    - **search_fields method adds a search criterea to the top of the class/table on the admin panel. in this instance, this search field will search the title and content values of the table. but other values could be searched as well by adding them to the list. The search_fields structures its search criteria like so:**
        -  WHERE (title ILIKE 'input value' OR content ILIKE 'input value')
    documentation: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields

Using these same methods, create a class for comments, in admin.py:
- add the @admin.register(Comment) tag before adding the new class
- add the class CommentAdmin(admin.ModelAdmin)
- give it a list_display variable with the tuple value of
    - list_display = ('name', 'body', 'post', 'created_on', 'approved')
- give it a list_filter variable with the tuple value of
    - list_filter = ('approved', 'created_on')
- give it a search_fields variable with the list value of
    - search_fields = ['name', 'email', 'body']

The admin panel (once refreshed) should now display another blog table for comments. with all of the above features

by default, the dhango admin panel will give a dropdown called "actions" which lets the admin delete items. however, we can add more functions to this dropdown witht he variable "actions".

the variable takes a list of function names as its arguments
- add the following code to the comments class:
    - actions = ['approve_comments']
- below the variable, leave one blank line then add the function approve_comments with self, request and queryset as its arguments:
    - def approve_comments(self, request, queryset):
        - queryset.update(approved=True)
    here, the function takes itself, and targets the queryset given as a request to update the item
    the one line updates the value of approved to True, then the request argument makes the request to the database to change the spproved status' value in the table.

with this completed, the 3 user stories currently in-progress are now complete

___

## Creating Views

The next step is to create the views of the blog.
to do this, the following needs to be achieved:
1. code for a view needs to be written
2. create a template that can render the view
3. connect it all up in the urls.py file

# [![Lesson 7: Creating the views](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/LP-glKOWpi8)

### step 1. code for a view needs to be written
- first, import the generic library from views in Django
    - from django.views import generic
- next, the created Post model needs to be imported from models.py
    - from .models import Post
- create a class called PostList that takes a method called ListView from the imported generic library as an argument
    - this will allow the use of predefined functions in the generic > ListView import, such as a pagination function
- the model variable is set as the Post model that has imported at the top of the file
- the class will be a queryset of all the objects inside the Post table, also, they will be only shown if they have been approved, so the filter method will be applied with the argument of status=1, because we defined in models.py that if the STATUS variabe == 1 it means the post has been published
- we'll tie the index.html template to this class too as its rendering template
- finally, the list will paginate itself by 6 posts per page with the paginate variable:
    - class PostList(generic.ListView):
        - model = Post
        - queryset = Post.objects.filter(status=1).order_by('-created_on')
        - template_name = 'index.html'
        - paginate_by = 6

### step 2. create a template that can render the views

- this project provided base templates for:
    - base.html
    - index.html
    - post_detail.html
    - style.css
- html templates can be found for reference, here:
    - https://github.com/Code-Institute-Solutions/django-blog-starter-files/tree/master/templates
- css template can be found here:
    - https://github.com/Code-Institute-Solutions/django-blog-starter-files/tree/master/static/css

### step 3. connect it all up in the urls.py file

# [![Lesson 8: Creating the first view](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/LP-glKOWpi8)