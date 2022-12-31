# I think, Therefore I Blog Django Project

## Set Up
* Note: Closely follow the [Django Notes](https://github.com/Grawnya/django-notes) repo, as it outlines how to start up the project and provides more primary info on Django & it's nuances.

#### Commands:
* `pip3 install 'django<4' gunicorn`
* `pip3 install dj_database_url==0.5.0 psycopg2`
* `pip3 install dj3-cloudinary-storage`
* `pip3 freeze --local > requirements.txt`
* `django-admin startproject codestar .`
* `python3 manage.py startapp app_name`
* Then add the app name to the `settings.py` file in the `INSTALLED_APPS` list.

#### The migration:
* `python3 manage.py migrate`
* `python3 manage.py runserver` (to check site works)

#### Heroku & ElephantSQL DB:
* Create a Heroku App
* Create an ElephantSQL DB and copy the URL
* Create an `env.py` file and save both the Django secret key and DB URL values
* In `settings.py`, set the `SECRET_KEY` variable to the saved one in `env.py`.
* Also comment out the `DATABASE_URL` variable, creating a copy of one, equal to a dictionary where `default : dj_database_url.parse(os.environ.get('DATABASE_URL'))`
* `python3 manage.py migrate` (to migrate your database structure to the newly-connected ElephantSQL DB)
* Add both the `DATABASE_URL` and `SECRET_KEY` to your Heroku project's config vars, as well as `PORT` is 8000.

#### Cloudinary:
* Create an account and copy the URL from your dashboard.
* Create a variable called `CLOUDINARY_URL` and set it to the URL in both the `env.py` file and the Heroku config vars.
* Also create a config var called `DISABLE_COLLECTSTATIC` and set it to 1, to ensure the deployment does not fail - but remove it when deploying the final project at end.
* In `settings.py`, in the `INSTALLED APPS` list, add `cloudinary_storage` above `django.contrib.staticfiles` and put `cloudinary` just above any self created apps.
* Under `STATIC_URL` add `STATICFILES_STORAGE` equal to `Cloudinary_storage.storage.StaticHashedCloudinaryStorage`, which connects to the `cloudinary_storage` app.
* Add also `STATICFILES_DIRS` and `STATIC_ROOT`, which are the locations of where the files will be stored and the suitable root, but the latter will not be used in this project.
* Similarly, create the variables `MEDIA_URL` and `DEFAULT_FILE_STORAGE` which is specifically set to `cloudinary_storage.storage.MediaCloudinaryStorage`.
* Add a `TEMPLATES_DIR` variable under the `BASE_DIR` to connect to the templates folder and within the `TEMPLATES` variable, add the `TEMPLATES_DIR` value to the list of the `APP_DIRS` value.
* Before deployment, set the `ALLOWED_HOSTS` variable to `the_name_of_heroku_app.herokuapp.com` and add `localhost` to the list so you can still work on the project locally.

#### Final Steps:
* Create `static`, `templates` and `media` folders at the top level as mentioned in the `settings.py` file.
* Create a Procfile with the command `web: gunicorn project_name_of_folder.wsgi` and commit the project to ensure it is up-to-date.

## Models:
* Go to the `models.py` file within the project folder.
* Open it up and import `from django.contrib.auth.models import User` to import user login data created later within a model. 
* Also import `from cloudinary.models import CloudinaryField` to import the Cloudinary Field for a featured image.
* To determine the post status, create a `STATUS` tuple equal to `((0, "Draft"), (1, "Published"))` so the programme knows the status of a user-created article in this case.
* Create classes for the tables representing a post's contents, inheriting from `models.Model`. See the [Django Notes](https://github.com/Grawnya/django-notes) repo for more details and the samples in [models.py](https://github.com/Grawnya/I-think-therefore-I-blog/blob/main/blog/models.py).
* Following the completion of the models, in the terminal place `python3 manage.py makemigrations` and add `--dry-run` beforehand if you want to check you are correct with spelling etc.
* Then make the final migrations by `python3 manage.py migrate`.

## Admin

#### Creating a Superuser:
* Type into the terminal `python3 manage.py createsuperuser` and you will be prompted to enter a username, email and password twice.
* Run the server using `python3 manage.py runserver` and at the end of the link, put `/admin` which loads the admin page and asks the user to login.

#### Adding Models to the Admin Page:
* Go to `admins.py` in the app folder and import the required model class.
* Include the line `admin.site.register(class_name)` to add the model to the admin page.

#### WYSIWYG:
* WYSIWYG (pronounced wiz-ee-wig) is a type of editing software that allows users to see and edit content in a form that appears as it would when displayed on an interface, webpage, slide presentation or printed document. WYSIWYG is an acronym for "what you see is what you get.
* In this project, summernote will be used whihc can be downloaded in the terminal with `pip3 install django-summernote` and added to the requirements.txt document (`pip3 freeze --local > requirements.txt`).
* Add the `django_summernote` app to the `INSTALLED_APPS`variable within `settings.py`. Note that it was installed with a `-`(hyphen), but the application is added with an `_`(underscore). This is common practice with Django libraries.
* Go to `urls.py` and import the `include` method within the `django.urls` library.
* Add a path with the `path`= `'summernote/'` and the `Pattern` = `include('django_summernote.urls')`, which will automatically register all summernote urls.

#### Summernote Fields:
* We need to tell our admin panel which field  we want to use Summernote for. So basically we're going to say that our a specific field, which is stored as a text field in the database is going to be a Summernote field e.g. `content` in this example.
* Within `admin.py`, add `from django_summernote.admin  import SummernoteModelAdmin` and create a new class that's going to inherit from  `SummernoteModelAdmin`.
* `summernote_fields = ('content')` using the `'content'` example uses the blog content from the model, which is a Django text field that we want to use a summernote for.
* Remove the `admin.site.register(Post)` line and add a decorator to the top of the new class instead `@admin.register(name_of_model)` which registers both the model and the class. The former line is removed as it only takes in 2 arguments so gets quite full quickly and the decorator is more pythonic.
* As more apps have been installed, you have to migrate again - `python3 manage.py migrate`.

#### Creating Slug Fields Automatically:
* Use the `prepopulated_fields` property, specifically designed for generating slug fields, which calls a bit of JavaScript that formats and populates the slug field.
* To use it, we pass in a dictionary that maps the field names to the fields that we want to populate from. This is done within the class in `admin.py` and is set to `{'slug': ('title',)}`, which automatically generates the `slug` when the user types out the `title`.
* You can also add `list_filter = ('status', 'created_on')`, which creates a filter feature based on either the status or the date when the post was created i.e. list all the model attributes that you want to filter by here .

#### Formatting Post Filters and Actions:
* Use `list_display = ('title', 'slug', 'status', 'created_on')` to display all the posts in a table, showing the mentioned category values.
* `search_fields` is used to enable a search box on the admin change list page. This should be set to a list of field names that will be searched whenever somebody submits a search query in that text box.
* To add an action such as an approval action to the admin site, you can use the `actions` built-in feature, which allows the user to specify different actions that can be performed from the action drop-down box. The default action is just to delete the selected items.
* Set `actions = ['function_of_approval']` and declare the function below e.g. one for `approve_comments` which allows the admin to set the value as `True`.
\
&nbsp;
`def approve_comments (self, request, queryset):`
\
&nbsp;
    `queryset.update(approved=True)`

## Views

#### Class-based Views:
* Class-based views allow us to make code that's reusable i.e. one view can inherit from another, which is not possible with standard function-based views. That means we can make use of some of the built-in features with Django, such as generic views.
* Each time we create a new view, we need to do three things - Create the view code, create the view template and connect up the urls file.
* Import a generic view by using `from django.views import generic` and create a class which inherits `generic.ListView`.
* Set the variable `model` to the import model and set the variable `queryset` to a suitable set, which is used to render the list of instances related to the class e.g. `queryset = Post.objects.filter(status=1).order_by('-created_on')`.
* `status` can be either `0` (draft) or `1` (published). Therefore, `1` is selected as we only want the user to see the published posts.
* `template_name` refers to the html page that should be loaded.
* `paginate_by` refers to the number of instances shown e.g. 6 posts.
* This creates the view code and then create html files within the `templates` folder to render the view, along with creating a CSS folder wihtin the `static` folder, creating a suitable `style.css` file within.

#### Altering Templates based on View
* Within a page, if you want to only show a certain number of elements per row e.g. 3 posts in a row, you can use the built-in loop counter e.g. `{% if forloop.counter|divisibleby:3 %}`.
* Within an instance or post in this example, if an image is not provided, as noted by `if "placeholder" in post.featured_image.url` above an image, then a default image is used.
* If pagination is used i.e. if a page only shows X options and more exist, then we want them to exist on another page, a `if is_paginated` loop has to be created before the final closing `div` tag at the bottom of the required page. 
* The pagination block of code in this project is boilerplate code and therefore, can be used in your project.

#### Connecting up the URLs
* Create a file called `urls.py` within the app folder - namely `blog` in this example.
* Import views and path and create the `urlpatterns` variable.
* If the path is empty, this suggests that it is the home page. Similar to previously creating paths, due to using class based views, `.as_view()` has to be placed at the end of the named class view.
* Open up the `urls.py` file for the overall project to connect the app's urls