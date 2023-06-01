## when starting the project
___

## Create an empty project

Check what version of python you are running with the following terminal command:
- `ls ../.pip-modules/lib`

# [![Lesson 1: Creating an empty django project](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/MagEuw9Q4T4)
### install
The following 3 packages / libraries are the first things to install when creating the project
- django and green unicorn (unicorn)
    - for walkthrough: package used:
        -   `pip3 install 'django<4' gunicorn`
    - modern ver:
        -   `pip3 install django gunicorn`
- postgres, dj database and psycopg2
    - for walkthrough: package used:
        - `pip3 install dj_database_url==0.5.0 psycopg2`
    - modern ver:
        - `pip3 install dj_database_url psycopg2`
- Coudinary
    - `pip3 install dj3-cloudinary-storage`

### create requirements.txt
- `pip3 freeze --local > requirements.txt`

### create a Django project in the workspace
- `django-admin startproject NAMEOFTHEPRJECT .`

### create the app within the project
-   `python3 manage.py startapp NAMEOFTHEAPP`
- the APPNAME then needs to be listed in the INSTALLED_APPS list variable in NAMEOFTHEPROJECT/settings.py

### migrate new changes to the newly created database
- `python3 manage.py migrate`

### check it runs
- `python3 manage.py runserver`

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
```python
import os
os.environ["DATABASE_URL"] = "ELEPHANTSQL DATABASE LINK"
os.environ["SECRET_KEY"] = "copy in a secret key or make one"
```

add these values to the heroku app as config vars under settings
| Key          | Value                     |
| -----------: | :------------------------ |
| `DATABASE_URL` | ELEPHANTSQL DATABASE LINK |
| `SECRET_KEY`   | SECRET KEY FROM ENV.PY    |
| `PORT`         | `8000`                      |

___

## link up env.py and database to project
add the following code under the imports at the top of settings.py
``` py
import os
import dj_database_url
if os.path.isfile('env.py'):
    import env
```

change the value of SECRET_KEY in settings.py to:
``` py
SECRET_KEY = os.environ.get('SECRET_KEY')
```

comment out the existing DATABASES sqlite3 database variable and add the following code:
``` py
DATABASES = {
    'default': dj_database_url.parse(os.environ.get("DATABASE_URL"))
}
```

with everything connected, migrate it all into manage.py
- `python3 manage.py migrate`

you can check this worked by going to ElephantSQL, going to the db, clicking the browser tab on the left and check under table queries to see if tables have been populated

___

## Deploying to Heroku

# [![Lesson 1a: deploying the empty project](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/Qhypx3Z2Heg)

1. create a cloudinary account
2. copy environment variable API key given on the cloudinary dashboard
3. paste it into env.py as an environment variable:
    -   ``` py
        os.environ["CLOUDINARY_URL"] = "CLOUDINARYAPIKEY"
        ```
4. add this new environment variable to the Heroku config vars
5. add a temporary config var to heroku:
    - `DISABLE_COLLECTSTATIC` : `1`
        - to mitigate having no static files yet for the time being
        - is removed when coming to final deployment
6. `'cloudinary_storage'` now needs to be added to `INSTALLED_APPS` in settings.py
    - this needs to be added ABOVE `'django.contrib.staticfiles'`
7. add `'cloudinary'` to `INSTALLED_APPS` in settings.py
    - this goes below `'django.contrib.staticfiles'`, but above our newest `'NAMEOFAPP'`
8. set your static files up to be linked with cloudinary storage:
    - under `STATIC_URL` in settings.py, add:
        ``` Python
        STATICFILES_STORAGE = 'cloudinary_storage.storage.StaticHashedCloudinaryStorage'
            # tells app to use cloudinary as storage for static files
        STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
            # makes our base directory at the top of the file static
        STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
            # not actually needed for this project but it is good practice to set
        ```
9. set up media storage
    - under the `STATIC` code written above, add:
        ``` py
        MEDIA_URL = '/media/'
        DEFAULT_FILE_STORAGE = 'cloudinary_storage.storage.MediaCloudinaryStorage'
            # allows the storing of media and images on cloudinary
        ```
10. add the directory for the app's templates:
    - under BASE_DIR, add
        ``` py
        TEMPLATES_DIR = os.path.join(BASE_DIR, 'templates')
        ```
    - in settings.py navigate to `TEMPLATES` and change the value of the `'DIRS` Key to:
        - `[TEMPLATES_DIR],`
11. Add allowed hosts for heroku deployment
    - add your heroku app link (the link used when deployed) to the value of `ALLOWED_HOSTS` in settings.py
        ``` py
        ALLOWED_HOSTS = ['HEROKU/APP/LINK.com', 'localhost']
        ```
        - also add local host so project can still be run from github
12. create the following top-level folders in the project:
    - media
    - static
    - templates
13. create a Heroku Procfile
    - add a new file, call it `Procfile`
    - add `gunicorn` to `Procfile`
        ``` 
        web: gunicorn appname.wsgi
        ```
        - "`web`" indicates that this app handles http traffic
        - "`wsgi`" stands for Web Services Gateway Interface server
14. Deploy to Heroku
    - add, commit, push everything
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
   -    ``` py
        from django.contrib.auth.models import User
        ```
2. import cloudinary to get featured images linked to database
    -   ``` py
        from cloudinary.models import CloudinaryField
        ```
3. create a tuple variable with the name "STATUS" with the value of (0, "draft), (1, "Published")
    -   ``` py
        STATUS = ((0, "Draft"), (1, "Published"))
        ```
4. begin creating the actual tables within python classes:
    -   ``` py 
        class Post(models.Model):
        # class imports the Model function from django's models library
            title = models.CharField(max_length=200, unique=True)
                # Makes a character field, single line with a max length of 200, specified as unique
            slug = models.SlugField(max_length=200, unique=True)
                # A slug is a string that can only include characters, numbers, dashes, and underscores. 
                # It is the part of a URL that [#14](https://github.com/Cal-Rex/django-blog-walkthrough/issues/14) 
                # identifies a particular page on a website, in a human-friendly form
            author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="blog_posts")  # noqa
                # - has a one-to-many relationship, so contains a foreignkey that can be used on other tables to identify it
                # - on_delete=models.CASCADE means that if an author is deleted, 
                #   all related items oertaining to blog_posts wil be deleted__
            updated_on = models.DateTimeField(auto_now=True)
                # datetime field that is defaulting to the current date and time
            content = models.TextField()
                # text field for content input
            featured_image = CloudinaryField('image', default='placeholder')
                # field that mkaes use of the importing of cloudinary, so we can import images from our cloudinary db into the project
            excerpt = models.TextField(blank=True)
                # a field for an excerpt of the full post text
            created_on = models.DateTimeField(auto_now_add=True)
                # same as updated on
            status = models.IntegerField(choices=STATUS, default=0)
                # integer field that picks from the choices given in the STATUS variable, 
                # values in in the STATUS tuple above determine the choices
            likes = models.ManyToManyField(User, related_name='blog_likes', blank=True)
                # Many to Many field, which links related tables with the name blog_data
            
            class meta:
                ordering = ['-created_on']
                    # structures entries in descending order

                def __str__(self):
                    return self.title
                        # this method returns the string definition of an object, 
                        # it has to target itself or it doesn't work

                def number_of_likes(self):
                    return self.likes.count()
                        # will count the number of likes stored on the likes entry
        ```
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
    - `python3 manage.py makemigrations`
    - `python3 manage.py migrate`
    - if you need to double check the tables before you try to migrate them, you can do a dry run with:
        - `python3 manage.py makemigrations --dry-run`

___

## building the admin and admin panel, and customising content input field on blog posts
# [![Lesson 5: Creating the admin panel](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/er5IKknKxoQ)

admin : 
| -------- | ----- |
| Username | admin |
| Password | admin |

1. we create an admin user using the terminal:
    - `python3 manage.py createsuperuser`
    - the terminal will then prompt you for username, email address and pw
        - pw wont be echod back to screen so you want be able to see what you're typing
2. admin page can be accessed by adding "/admin" to the end of the workspace url
3. add the Post model created in models.py to the Admin panel
    - in admin.py import the "Post" class from ".models"
        -   ``` py
            from .models import Post
            ```
    - add the following code:
        -   ``` py
            admin.site.register(Post)
            ```
4. add a WYSIWYG editor for the post
    - intall the summernote library
        - `pip3 install django-summernote`
        - documentation for summernote: https://summernote.org/
    - update the requirements.txt as we have installed a new library
        - `pip3 freeze --local > requirements.txt`
    - add summernote to list of `INSTALLED_APPS` in settings.py
        - use the syntax: `django_summernote`
    - the URLs now need to be set up, navigate to urls.py
        - on the import line for django.urls, add "import" to the list of libraries being imported
            - line should now look like this:
                -   ``` py
                    from django.urls import path, include
                    ```
        - in the `urlpatterns[]` variable, add a new path for summernote:
            -   ``` py
                path('summernote/', include('django_summernote.urls')),
                ``` 
                - the include library that was just imported is used here
    - the admin panel now needs to be told what we want to use the summernote functionality for, which in this case, its to be used for the content field instead of a basic TextField
        - at the top of admin.py, import SummerNoteModelAdmin from django_summernote.admin
            -   ``` py
                from django_summernote.admin import SummerNoteModelAdmin
                ```
            - with the new import, create a class called PostAdmin that inherits from SummerNoteModelAdmin, under the line:
                -   ``` py
                    admin.site.register(Post)
                    ```
            - inside the class add the following:
                -   ``` py
                    summernote_fields = ('content')
                    ```
                    - this links the summernote functionality to fields with the name 'content'
            - above the class replace the following codeline:
                -   ``` py
                    admin.site.register(Post)
                    ```
                    - we are replacing this because this method only allows for 2 arguments/parameters, so it fills up too quickly / has reduced functionality
            - with:
                -   ``` py 
                    @admin.register(Post)
                    ```
                    - this class decorator will register both the Post model and the Post admin class with the admin site.
        - migrate all these changes
            - because we havent made any structural database changes, the makemigrations command doesnt need to be run, just the migrate command.
                - `python3 manage.py migrate`
        - run the project
            - `python3 manage.py runserver`
        - then add `/admin` to the url once loaded in preview (the main site has no page so will come up with a debug screen)
        - if you try to add a new post you can see that summernote has overridden the content box with a text editor
        - make a pilot test post

___

## creating the admin panel, part 2
# [![Lesson 6: Creating the admin panel](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/JGt1p8JhPyY)

When we enter the post title, we want the  slug field to be generated automatically.
there is a variable in django that uses javascript to populate fields with prepopulated data.

in admin.py under the PostAdmin(SummernoteModelAdmin) class, add:
-   ``` py
    prepopulated_fields = {'slug': ('title',)}
    ```
    - the variable contains an object with a KVP, the key being the 'slug' field, and the value will be 'title' wrapped inside a tuple (as the value could contain more than one parameter)

then, we can add a post filter function on the admin panel bu adding another line of code to the class:
-   ``` py
    list_filter = ('status', 'created_on')
    ```
we can then add other functions from the django documentation that expand the details given of posts on the admin panel:
-   ``` py
    list_display = ('title', 'slug', 'status', 'created_on')
    ```
    - The list display method from django allows for table elements to be listed in a tuple as its parameter, then on the admin panel, these details will be displayed top level for the class/table
    - documentation: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list_display
-   ``` py 
    search_fields = ['title', 'content']
    ```
    - `search_fields` method adds a search criterea to the top of the class/table on the admin panel. in this instance, this search field will search the title and content values of the table. but other values could be searched as well by adding them to the list. The search_fields structures its search criteria like so:
        -  `WHERE (title ILIKE 'input value' OR content ILIKE 'input value')`
    documentation: https://docs.djangoproject.com/en/3.2/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields

Using these same methods, create a class for comments, in admin.py:
- add the `@admin.register(Comment)` tag before adding the new class
- add the `class CommentAdmin(admin.ModelAdmin)`
- give it a list_display variable with the tuple value of
    -   ``` py
        list_display = ('name', 'body', 'post', 'created_on', 'approved')
        ```
- give it a list_filter variable with the tuple value of
    -   ``` py
        list_filter = ('approved', 'created_on')
        ```
- give it a search_fields variable with the list value of
    -   ``` py
        search_fields = ['name', 'email', 'body']
        ```

The admin panel (once refreshed) should now display another blog table for comments. with all of the above features

by default, the dhango admin panel will give a dropdown called "actions" which lets the admin delete items. however, we can add more functions to this dropdown with the variable "actions".

the variable takes a list of function names as its arguments
- add the following code to the comments class:
    -   ```py
        actions = ['approve_comments']
        ```
- below the variable, leave one blank line then add the function `approve_comments` with `self`, `request` and `queryset` as its arguments:
    -   ``` py
        def approve_comments(self, request, queryset):
            queryset.update(approved=True)
        ```
    here, the function takes itself, and targets the queryset given as a request to update the item
    the one line updates the value of approved to True, then the request argument makes the request to the database to change the approved status' value in the table.

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
    -   ``` py 
        from django.views import generic
        ```
- next, the created Post model needs to be imported from models.py
    -   ```py
        from .models import Post
        ```
- create a class called PostList that takes a method called ListView from the imported generic library as an argument
    - this will allow the use of predefined functions in the generic > ListView import, such as a pagination function
- the model variable is set as the Post model that has imported at the top of the file
- the class will be a queryset of all the objects inside the Post table, also, they will be only shown if they have been approved, so the filter method will be applied with the argument of `status=1`, because we defined in models.py that `if` the `STATUS` variabe `== 1` it means the post has been published
- we'll tie the index.html template to this class too as its rendering template
- finally, the list will paginate itself by 6 posts per page with the paginate variable:
    -   ``` py
        class PostList(generic.ListView):
            model = Post
            queryset = Post.objects.filter(status=1).order_by('-created_on')
            template_name = 'index.html'
            paginate_by = 6
        ```

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

# [![Lesson 8: Creating the first view](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/LP-glKOWpi8)

utilise django html code strings in index.html to generate a for loop that displays published posts:
- !! Remember !! {{ Double curly braces }} inserts direct value of code, {% percentage wrapped in  curly braces %} runs python/django functions
- in index.html, in the column that is to display the blog posts, add a django for loop
    - Inside the forloop code, add an if statement that takes action every time the loop counts 3 posts (basically target the counter variable, that goes up by 1 for every "post" in "post_list" and then check if that number is divisible by 3)
    - the {% if "placeholder" ... %} line checks to see if the placeholder image is being used by checking the url used for the image, if true, it will display the placeholder image, if not, it will display the designated deatured img by its url
        -   ``` html
            {% for post in post_list %}
                    <div class="col-md-4">
                        <div class="card mb-4">
                            <div class="card-body">
                                <div class="image-container">
                                    {% if "placeholder" in post.featured_image.url %}
                                    <img src="https://codeinstitute.s3.amazonaws.com/fullstack/blog/default.jpg" class="card-img-top">
                                    {% else %}
                                    <img src="{{ post.featured_image.url }}" class="card-img-top">
                                    {% endif %}
                                    <div class="image-flash">
                                        <p class="author">Author: {{ post.author }}</p>
                                    </div>
                                </div>
                                <a href="#" class="post-link">
                                    <h2 class="card-title">{{ post.title }}</h2>
                                    <p class="card-text">{{ post.excerpt }}</p>
                                </a>
                                <hr />
                                <p class="card-text text-muted h6">{{ post.created_on }} <i class="far fa-heart"></i> {{ post.number_of_likes }}</p>

                            </div>
                        </div>
                    </div>
                    <!-- if the counter is divisible by 3, this loop breaks the div and creates a new div/row -->
                    {% if forloop.counter|divisibleby:3 %}
                    </div>
                    <div class="row">
                    {% endif %}
                    {% endfor %}
            ```

### step 3. connect it all up in the urls.py file
first, create a new file and name it urls.py in the blog folder

inside the new file, import the views.py file (. stands for top level of project):
-   ``` py
    from . import views
    ```

Then, from the django library, import the path functionality from the urls library, so we can use the same path functions that are being used within urls.py file in the codestar folder.
-   ``` py
    from django.urls import path
    ```

with those in place, we now need to create a url pattern for the home page.
- create a list variable named `urlpatterns`
- inside the list, for the first item add a `path` method, its first argument being a `blank string in single quotes`, the second argument should target the `PostList` `class` inside views.py, which has been imported, which in turn is then specified to render as a view by appending the `.as_view()` function to the value, then give it third argument which establishes a `name` variable with the value of `'home'`
    -   ``` py
        urlpatterns = [
            path('', views.PostList.as_view(), name='home')
        ]
        ```

Once that's been created, we then need to import this to the codestar urls file. in codestar > urls.py add a third path to `urlpatterns`:
-   ``` py 
    path('', include('blog.urls'), name='blog_urls')
    ```
    - the include() function is specified by django as such:  
        - "it chops off whatever part of the URL matched up to that point and sends the remaining string to the included URLconf for further processing."  - stack overflow further clarified - it takes the remaining string after '' and allows mapping of patterns listed in blog.urls to certain views based on the urls defined in that file instead.

BUGFIX - at this point i ran into an error when tyring to load the page.
```
TemplateDoesNotExist at /
    index.html, blog/post_list.html
    Request Method:	GET
    Request URL:	http://localhost:8000/
    Django Version:	3.2.19
    Exception Type:	TemplateDoesNotExist
    Exception Value:	
    index.html, blog/post_list.html
    Exception Location:	/workspace/.pip-modules/lib/python3.8/site-packages/django/template/loader.py, line 47, in select_template
    Python Executable:	/home/gitpod/.pyenv/versions/3.8.11/bin/python3
    Python Version:	3.8.11
    Python Path: ['/workspace/django-blog-walkthrough',
        '/home/gitpod/.pyenv/versions/3.8.11/lib/python38.zip',
        '/home/gitpod/.pyenv/versions/3.8.11/lib/python3.8',
        '/home/gitpod/.pyenv/versions/3.8.11/lib/python3.8/lib-dynload',
        '/workspace/.pip-modules/lib/python3.8/site-packages',
        '/home/gitpod/.pyenv/versions/3.8.11/lib/python3.8/site-packages']
```
This is because i was missing the value of `TEMPLATES_DIR` from the `'DIRS'` variable inside the `TEMPLATES` list variable in settings.py
- here is the explanation from the tutor that helped: 
    - You are missing the directory `TEMPLATES_DIR` from line 70. This is how to tell django to connect all of your templates into the directory so that are all accessible and it's why it was giving us the error message that it couldnt find anything at all

The page should now load, and you should be able to see template test posts originally posted on the admin panel
___

## creating a view for post details

the same 3 steps now need to be repeated for the Post details view

1. code for a view needs to be written
2. create a template that can render the view
3. connect it all up in the urls.py file

navigate to views.py
import `View` from `django.views`, `View` should just be appended after `generic` on the `from`|`import` line. for example:
-   ``` py
    from django.views import generic, View
    ```
- repeat this process, adding `get_object_or_404` to the `django.shortcuts` imports
-   ``` py
    from django.shortcuts import render, get_object_or_404
    ```

then add a `class` called `PostDetail` with the argument of(`View`):
-   ``` py
    class PostDetail(View):

        def get(self, request, slug, *args, **kwargs):
            # the function target's itself (the PostDetail View)
            # it takes the request function as an argument to be used later
            # it takes the slug value of the target post (reasoning below)
            # *args allows us to pass a variable number of non-keyword arguments to a Python function
            # **kwargs allows us to pass a variable number of keyword arguments to a Python function
            queryset = Post.objects.filter(status=1)
                # filter through all of the published posts by doing a queryset on all posts 
                # and filter to return only the ones with  a status of 1
            post = get_object_or_404(queryset, slug=slug)
                # as the slug is a unique identifier for a post, we can use the get_object_or_404 function
                # from the library to get an object within the queryset (1st argument)
                # that has the matching slug value of the slug that was parsed in as the initial argument
                # at the top of the function
            comments = post.comments.filter(aproved=True).order_by('created_on')
                # with the selected post established in the variable, 
                # data from inside the post can now be accessed for use by functions
                # here the comments attached to that post can be directly targeted and filtered/ordered 
                # ordering by "created_on" orders posts by oldest first, so it flows like a convo
                # "-created_on" orders by newest first
            liked = False
            if post.likes.filter(id=self.request.user.id).exists():
                liked = True
            # the liked variable coupled with this if statement checks and delivers an action if the
            # OP liked the post
            # it does so by created a boolean variable with the value of false
            # then, in the if statement, targets the likes variable inside the post variable, 
            # then for its argument, check to see if the viewer's id matches that of the id sent in the request. 
            # if it exists, then the liked variable is set to True
            
            return render(
                request,
                "post_detail.html",
                {
                    "post": post,
                    "comments": comments,
                    "liked": liked
                }
            )
    ```

the difference between `class` based views and `function` based views:
|               Class-based view example                                         |       function-based view example       |
| ------------------------------------------------------------------------------ | --------------------------------------- |
|                                                                                |                                         |
| class PostDetail(View):                                                        | `def add_item(request):`                |
|                                                                                | `      if request.method == 'POST':`    |
|      def get(self, request, slug, *args, **kwargs):                            | `      form = ItemForm(request.POST)`   |
|          queryset = Post.objects.filter(status=1)                              | `        if form.is_valid():`           |
|          post = get_object_or_404(queryset, slug=slug)                         |                                         |
|          comments = post.comments.filter(aproved=True).order_by('created_on')  |                                         |
|          liked = False                                                         |                                         |
|          if post.likes.filter(id=self.request.user.id).exists():               |                                         |
|              liked = True                                                      |                                         |
|          return render(full code a above)                                      |                                         |

While function-based views use `if` statements to check the `request`, class-based views use the function names inside the actual class to define the reuest method, and what happens when that request is made.

With the view now built, the template needs to be filled out and the urls.py file needs to have a path specified to hook everything up.

- post detail html template: https://github.com/Code-Institute-Solutions/django-blog-starter-files/blob/master/templates/post_detail.html

Using the post variable from PostDetail inside views.py, variable names specified inside the Post table inside models.py can be accessed when writing in django/python objects into the html template:

``` html
<h1 class="post-title">
    {{ post.title }}
</h1>
<!-- Post author goes before the | the post's created date goes after -->
<p class="post-subtitle">{{ post.author }} | {{ post.created_on }}</p>
```

certain filters are used within the inline djano code in this template:
``` html
<p class="card-text ">{{ post.content | safe }}</p>
```
This flag tells Django that if a “safe” string is passed into your filter, the result will still be “safe” and if a non-safe string is passed in, Django will automatically escape it, if necessary. You can think of this as meaning “this filter is safe – it doesn't introduce any possibility of unsafe HTML.”

``` html
<!-- The body of the comment goes before the | -->
                    {{ comment.body | linebreaks }}
```
The linebreaks filter replaces line breaks with <br> and double line breaks with <p>. See also the linebreaksbr filter. Syntax. {{ value|linebreaks }}

with the template written and the view established. we now need to link it up with urls.py:
add the following path to urls.py in **blog**, _not codestar_.
```py
path('<slug:slug>/', views.PostDetail.as_view(), name='post_detail'),
```
- it is worth noting here that when a python object is being passed into the url, its needs to go in angle brackets!
- upon receiving a slug, the path will use the second argument to navigate into views.py, find the PostDetail class and render it as a view.
    - the first mention of slug in the url path i called a path converter, i basically takes the second value and turns it into a slug field, that django can understand
    - the second slug is a keyword name for the variable we are targetting, which in this instance is called slug

finally, with everything created and hooked up, all that is needed is to add the requisite url and ontrol statement to index html.
``` html
<a href="{% url 'post_detail' post.slug %}" class="post-link">
    <h2 class="card-title">{{ post.title }}</h2>
    <p class="card-text">{{ post.excerpt }}</p>
</a>
```
___

## Adding authentication

# [![Lesson 9: Adding Authentication](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/LP-glKOWpi8)

Django has built-in user authentication - as demonstrated when creating a superuser, however, for this proejct the django-allauth module is going to be used.
the reasons are as follows:
- adds functionality to send password and account confirmation emails 
- enforces password complexity
- provides single sign-on using google or facebook
- provides stock sign in / sign up pages (no CSS)  

Full Django Allauth documentation:
- https://django-allauth.readthedocs.io/en/latest/

to get started, install django-allauth in the terminal:
- `pip3 install django-allauth`
    - bear in mind, requirements.txt will need to be updated when installing a new package!
        - `pip3 freeze --local > requirements.txt`

with the package installed, the first step is to add allauth urls to the main urls.py file like what was done with summernote.
go to codestar > urls.py (NOT TO BE CONFUSED WITH blog > urls.py) and add the following path:
-   ``` py
    path('accounts/', include('allauth.urls')),
    ```

Next, settings.py needs to be updated to include allauth in the INSTALLED_APPS list, below `django.contrib.messages` / above `cloudinary_storage`:
-   ``` py
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    ```

with these libraries added, we also need to add a new variable to settings.py called `SITE_ID`, this is so that django can handle multipe sites from the one database. while this project only has one site using the databse, we still have to tell djago the site number. add the following variable beneath `INSTALLED_APPS` / above `MIDDLEWARE`:
-   ``` py
    SITE_ID = 1
    ```

login and logout urls also need to be added as variables here in the settings. add these variables beneath `SITE_ID`:
-   ``` py
    LOGIN_REDIRECT_URL = '/'
    LOGOUT_REDIRECT_URL = '/'
    ```

also, during the testing process, if you dont want to have to test with emails, you can turn this off by adding this variable to settings.py too:
-   ``` py
    ACCOUNT_EMAIL_VERIFICATION = 'none'
    ```

With the new package/feature added in the back end. the updates to the app need to be migrated again.
- `python3 manage.py migrate`

once successfully migrated, check the feature works by running the project:
- `python3 manage.py runserver`

add the following to the URL to see if the sign in/sign up feature has been implemented:
- `/accounts/signup`
- be mindful that you need to be logged out of admin otherwise this won't work.

with that functionality now working, we need to hook up the new sign in/ sign up pages with our actual site so users can access it properly. go to base.html and link up nav items.
- under the `{% if user.is_authenticated %}` string, add the following url strings to the relevant anchor tags:
    - logout: `{% url 'account_logout' %}`
    - Sign up: `{% url 'account_signup' %}`
    - login: `{% url 'account_login' %}`

with that implemented. Check it works.

test user created using site:

| Username  |  Password  |
| --------- | ---------- |
| test-user | Password+1 |

___

## Styling the Authenticaion process

# [![Lesson 10: Adding Authentication](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/IYlhc4c64U4)

before we proceed we need to double-check what version of python that we are using. you can do this with the following terminal command:
- `ls ../.pip-modules/lib`

Once we know what version we are running, we can use the following command to import all of the templates within python - the library sitting above our workspace - into the workspace so that we can tweak them to make them look how we want.
- `cp -r ../.pip-modules/lib/VERSIONSOFPYTHONINHERE/site-packages/allauth/templates/* ./templates`
- the command used for this project is:
    - `cp -r ../.pip-modules/lib/python3.8/site-packages/allauth/templates/* ./templates`
    - [`copy`] [`recursive`] [`fileplath`] [`destination for copy`]

This will create multiple directories in the templates folder. particularly, templates for the login/signup/logout pages in the `account` folder.

navigate to templates > account > login.html

At the top of the file, alter the {% extends %} tag so that this template extends from the already made base.html in the project. not `accounts/base.html`

next, this project doesnt use the social account part, so we are just going to comment it out for now.

We can now begin styling the html elements with bootstrap. use this quick emmet abbreviation to create
-   ``` html
    .container>.row>.col-md-8.mt-3.offset-md-2
    ```
    - This creates a container with a column div inside a row div with an offset of 2 so that the container is centred
    - Take the 3 closing `</div>` tags from this new container and cut/paste them at the bottom of the html file above the {% endblock %} tag so that all the form information is contained within the container

change the tag wrapping {% trans "sign in" %} from an `h1` to an `h3`

change the inner text inside {% blocktrans %} to:
``` html
Welcome back to the Code|Star blog. 
To leave the comment or like a post, please log in. If you
have not created an account yet, then
<a class="link" href="{{ signup_url }}">sign up</a>.
```
close this column and row off and create another row/column:
``` html
    </div>
</div>
.row>.col-md-8.mt-3.offset-md-2
<!-- dont forget to remove the 2 new closing tags from the mmet shortcut -->
```

contain the form elements inside this second row>column.

then, change the form button clas sstyles with some better bootstrap styling...
``` html
<button class="btn btn-signup right" type="submit">{% trans "Sign In" %}</button>
```

run the project and check the form is working properly.

If all goes to plan, copy/paste the new templates in for logout and signup:
- https://github.com/Code-Institute-Solutions/django-blog-starter-files/tree/master/templates/account

___

## Enable Commenting

# [![Lesson 11: enable commenting ](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/dm1MToEiXuw)
> At 0:50 in this video, Matt installs a package called django-crispy-forms into the walkthrough project.
> Since this video was made, a newer version of django-crispy-forms has been released which is automatically installed using the command from this video. To ensure you are working with the same package that Matt uses in this video, please use the following command to install crispy forms for this walkthrough:
- `pip3 install django-crispy-forms==1.14.0`

because we have added a new package for the tutorial, lets run the standard process of:
1. updating requirements.txt:
    - `pip3 freeze --local > requirements.txt`
2. add `'crispy_forms'` to `INSTALLED_APPS` in settings.py

in additon to these steps, also add an additional variable to settings.py to make sure that crispy uses bootstrap for its formatting:
-   ``` py
    CRISPY_TEMPLATE_PACK = 'bootstrap4'
    ```

with crispy forms set up, we now need to create a form class.
start by creating a new file in the `blog` folder called `forms.py`

in the new file, import the `Comment` class from models.py, and also `forms` from `django`
-   ``` py
    from .models import Comment
    from django import forms
    ```

Then, create a class called `CommentForm` with an argument that pulls the `ModelForm` template from the imported `forms` package:
-   ``` py
    class CommentForm(forms.ModelForm):
    ```

Inside this class, add a `Meta` class:
-   ``` py
    class Meta:
        model = Comment
        fields = ('body',) 
            # trailing comment needed here in the value, so that python recognises the value as a tuple and not a string
            # having a string value for the fields variable would throw an error
    ```
With the `CommentForm` class created, we now need to import it into our views.py file. add the following import command to the top of views.py:
-   ``` py
    from .forms import CommentForm
    ```

now that the form is successfully imported, it needs to be rendered as part of the view.
- inside the `PostDetail` Class, in the return statement at the bottom of the class, add the following to the 3rd argument/dictionary in the render method so that it looks like this:
    -   ``` py
        return render(
            request,
            "post_detail.html",
            {
                "post": post,
                "comments": comments,
                "liked": liked,
                "comment_form": CommentForm()
            }
        )
        ```
Next, in `post_detail.html`, add the django code in to implement the crispy forms functionality on the page. at the top of the file, add:
-   ```html
    {% load crispy_forms_tags %}
    ```

Then, we need to add the comment html to `post_detail.html`, in the last `<div>` with the class `card-body` (around line 79, bottom of the code):
-   ``` html
    {% if user.is_authenticated  %}

    <h3>Leave a comment:</h3>
    <p>Posting as: {{ user.username }}</p>
    <form method="post" style="margin-top: 1.3em;">
        {{ comment_form | crispy }}
        <!-- This renders the form using the crispy filter to make it look nice. -->
        {% csrf_token %}
        <!-- cross site request forgery is a method in which hackers try to attack sites and steal user data
        the csrf token is a built in method in django that prevents from such attacks, it must be added to all forms that are created for security purposes -->
        <button type="submit" class="btn btn-signup btn-lg">Submit</button>
    </form>
    {% endif %}
    ```

run the app now to see if everything is displaying as it should, though note: trying to submit a comment will throw an error at this stage.

___

## adding the POST method

# [![Lesson 12: adding commenting](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/K200vsthNQU)

Add a POST method to the `PostDetail` class.

Naturally, a POST method normally requires the same info as a GET method, so we can start by copying the existing `get` funtion in the `PostDetail` class, and then changing its name from `get` to `post`:
``` py
def post(self, request, slug, *args, **kwargs):
    queryset = Post.objects.filter(status=1)
    post = get_object_or_404(queryset, slug=slug)
    comments = post.comments.filter(approved=True).order_by('created_on')
    liked = False
    if post.likes.filter(id=self.request.user.id).exists():
        liked = True
    
    # add next piece of code below this example - here

    return render(
            request,
            "post_detail.html",
            {
                "post": post,
                "comments": comments,
                "liked": liked,
                "comment_form": CommentForm
            }
        )
```

Then, because we havent previously accounted for the new comment box on the post_details page when building the PostDetail class. We are going to need to add it in the space specified by the comment in the example above:
```py
    comment_form = CommentForm(data=request.POST)
```
This assigns the form data to a variable. the code here determines that the value of the data for `CommentForm` is retreieved when the `POST` `request` has been made.

With that in place, we now need to make sure the data passed into the `CommentForm` is valid, lest we create any errors or unsuitable data. underneath the `comment_form` variable, add an `if` statement to check that the form `is_valid()`. If true, the following commands should run:
``` py
    if comment_form.is_valid():
        comment_form.instance.email = request.user.email
        comment_form.instance.name = request.user.username
        comment = comment_form.save(commit=False)
        comment.post = post
        comment.save()
    else:
        comment_form = CommentForm()
```
- `instance` targets the that specific instance of `comment_form` meaning you can make a request of the data from a user making the post request in that `instance`
- at this point, we want to save what we have, but we dont want to commit/post any data to the server, as we want to make sure all the data is retrieved before it is saved and published., hense the use fo creating the `comment` variable, but attaching a `(commit=False)` argument to the `save` method. 
- now we link up the comment to its specified post, where the `post` variable the `comment` class should now share the value of the specified `post`
- with the `if` statement written out, the `else` statement needs to be applied to avoid an error being thrown. instead, the `comment_form` just reloads an instance of `CommentForm()`.


next, we add a new value into the `return render()` argument's dictionary variable. after `"comments": comments,` add `"commented": True`:
``` py
    return render(
            request,
            "post_detail.html",
            {
                "post": post,
                "comments": comments,
                "commented": True,
                "liked": liked,
                "comment_form": CommentForm
            }
        )
```

Because we have added a new KVP to the return render in the `POST` method, we will now need to amend the `GET` method. The `"commented"` KVP needs to be added into the same place in the `get` function, except this time its value needs to be `False`
``` py
    return render(
            request,
            "post_detail.html",
            {
                "post": post,
                "comments": comments,
                "commented": False,
                "liked": liked,
                "comment_form": CommentForm
            }
        )
```

With that added, lets add the approval function to the commenting process.

inside `post_detail.html` add the following code to above `{% if user.is_authenticated() %}`:
``` html
{% if commented %}
<div class="alert alert-success" role="alert">
    Your comment is awaiting approval
</div>
{% else %}
```
- The new `if` statement checks if the `"commented"` KVP from the `return` True, if it is, the user will get a note to say the comment is awaiting approval, otherwise it will just show the form.
- also, as we have just added a new `if` statement to the html, be sure to close it at the bottom of the code (so there should be 2 `endif`s now):
``` html
{% endif %}
{% endif %}
```

Test the project to see if the new functionality works
1. run the project in the local environment:
    - `python3 manage.py runserver`
2. Login
3. select a published post
4. make a comment and submit it
5. if woking correctly the comment box should say:
    - > Your comment is awaiting approval
6. Logout
7. append `/admin` to the url and login as admin
8. check the comment table to see the unpublished comment
9. approve it!

___

## Enable likes

# [![Lesson 13: enable likes ](http://img.youtube.com/vi/YOUTUBE_VIDEO_ID_HERE/0.jpg)](https://youtu.be/bNqsrk8x1TI)

to enable the likes, we are going to need another view. this means we need to follow the same 3-step process as before:

1. Create the code
2. Create a template to render the view
3. Connect up our URLs in the urls.py file

1. **create the view code**
    - in views.py create a new class-based view called PostLike
    - It should inherit from View
    - The method will be post and accept 3 parameters: self, request, and slug
        ``` py
        class PostLike(View):
        
        def post(self, request, slug):
        ```
    - before we can get this post function to do anything, it needs to know what it's targeting, so we make the variable `post` and apply the standard django `get_object_or_404` method as before. it's target object being the unique identifier of the post (which would be the `slug`). so we target the the table item within the `Post` database with the appropriate `slug`:
        ``` py
        class PostLike(View):
        
        def post(self, request, slug):
            post = get_object_or_404(Post, slug=slug)
        ```
    - now that we have targeted the specific post, we need to check if it has already been liked, and if it has already been liked by the user, that like needs to be removed if the like is selected/toggled
    - to do this, create an `if` statement that targets the `post`, its `likes` variable, and then `filter` through the likes by `id`, checking that it matches the `user` `id` making the `request` to see if it `exists()`
        - with that filter running as the `if` statement - if the user's `id` is found among the `likes`, the next line of code will `remove` the `likes` value of the `request`ed `user`. with no longer having a value, the `likes` variable will revert to its previously defined boolean value of `False` in the original `PostDetail` class.
        - the `else` statement will just do the same as above, except, instead of `remove`, the `add` method is used instead with the same parameters to apply the same logic in reverse.
        ``` py
        class PostLike(View):
            
            def post(self, request, slug):
                post = get_object_or_404(Post, slug=slug)
            
                if post.likes.filter(id=request.user.id).exists():
                    post.likes.remove(request.user)
                else:
                    post.likes.add(request.user)
        ```
    - now because we want the toggle to have an instantaneous response, we need to use another package that allows the value to be updated in real-time (without re-loading the whole page). we need to import a new function from django called `HttpResponseRedirect`, add it with the following `import` command at the top of the file:
        ``` py
        from django.http import HttpResponseRedirect
        ```
    - also, so that we can target URLs by the name we've given them in urls.py, we need to add the `reverse` method from the `django.shortcuts` library, the same place where we are pulling the `get_object_or_404` method from:
        ``` py
        # the following should be line-1 in the views.py file.
        from django.shortcuts import render, get_object_or_404, reverse
        ```
    - back down at the new `PostLike` class, we need to now add a return statement that takes advantage of our newly imported shortcut methods:
        ``` py
        class PostLike(View):
            
            def post(self, request, slug):
                post = get_object_or_404(Post, slug=slug)
            
                if post.likes.filter(id=request.user.id).exists():
                    post.likes.remove(request.user)
                else:
                    post.likes.add(request.user)
            
            return HttpResponseRedirect(reverse('post_detail', args=[slug]))
        ```
    - upon a successful iteration of the `PostLike` class, the `return`ing response from the class will reload the `post_detail` template tied to the specific `slug` of the instanced post, saving a complete refresh of all of the templates.

2. **Create a template to render the view**
    - go to the `post_detail.html` template
    - we now need to update the heart icon on the template to turn it into a button, so that it toggles when clicked, but only when an authenticated user is logged in
    - to do this, we need to create a form masquerading as a single button or icon.
    - first, in the column that displays `{{ post.number_of_likes }}`, add a django `if statement` at the top of the div to see if the viewing user is authenticated:
        ``` html
        <div class="col-1">
            <strong>
                {% if user.is_authenticated %}
                {% endif %}
            </strong>
        </div>
        ```
    - with that set up, inside the `if` statement we can add the form
    - the form now needs to first check if the boolean value is already set to display the correct type of icon on the button, so an additonal `if` / `else` statement needs to be nested into the initial `if user.is_authenticated` statement that checks if the `liked` variable of the instanced `PostDetail` has the boolean value of `True` 
    - also, a contingency needs to be put in plae if the user viewing the page is not an authenticated user, so an `else` statment needs to be added to the `if user.is_authenticated` to handle that potential outcome
    - finally, we will want to display the number of likes a post has next to the liked icon, which should give us the following result:
        ``` html
        <div class="col-1">
            <strong>
                {% if user.is_authenticated %}
                <form class="d-inline" action="{% url 'post_like' post.slug %}" method="POST">
                    <!-- action targets the PostLike template that has the matching post.slug -->
                    <!-- the PostLike class is linked up to the form by its return statement, and also in the urls.py file, giving it a path with the name of post_like which is shown below. -->
                    {% csrf_token %}
                    <!-- ALWAYS REMEMBER THE USE OF A CSRF TOKEN ON FORMS -->
                    {% if liked %}
                    <button type="submit" name="blogspot_id" value="{{post.slug}}" class="btn-like"><i class="fas fa-heart"></i></button>
                    {% else %}
                    <button type="submit" name="blogspot_id" value="{{post.slug}}" class="btn-like"><i class="far fa-heart"></i></button>
                    {% endif %}
                </form>
                {% else %}
                <span class="text-secondary"><i class="far fa-heart"></i></span>
                {% endif %}
                <span class="text-secondary"> {{ post.number_of_likes }}</span>
            </strong>
        </div>
        ```
3. **Connect up our URLs in the urls.py file**
    - create a path in blog > urls.py file in the `urlpatterns` list with the following prameters:
        - the path should be: `like/<slug:slug>`
        - the view should be `PostLike`, cast `as_view()`
        - The name needs to be `post_like`, the same as the url reference in both the form, and the `return` statement at the bottom of the `PostLike` class
        ``` py
        urlpatterns = [
        path('', views.PostList.as_view(), name='home'),
        path('<slug:slug>/', views.PostDetail.as_view(), name='post_detail'),
        path('like/<slug:slug>', views.PostLike.as_view(), name='post_like'),
        ]
        ```

Once implemented, test to see if it works.
- `python3 manage.py runserver`
- login as a user
- try liking something and see what happens