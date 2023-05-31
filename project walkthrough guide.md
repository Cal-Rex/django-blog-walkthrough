## when starting the project
___

## Create an empty project

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

with tat implemented. Check it works.