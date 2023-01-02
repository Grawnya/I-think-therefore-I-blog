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
\
&nbsp;
## Models:
* Go to the `models.py` file within the project folder.
* Open it up and import `from django.contrib.auth.models import User` to import user login data created later within a model. 
* Also import `from cloudinary.models import CloudinaryField` to import the Cloudinary Field for a featured image.
* To determine the post status, create a `STATUS` tuple equal to `((0, "Draft"), (1, "Published"))` so the programme knows the status of a user-created article in this case.
* Create classes for the tables representing a post's contents, inheriting from `models.Model`. See the [Django Notes](https://github.com/Grawnya/django-notes) repo for more details and the samples in [models.py](https://github.com/Grawnya/I-think-therefore-I-blog/blob/main/blog/models.py).
* Following the completion of the models, in the terminal place `python3 manage.py makemigrations` and add `--dry-run` beforehand if you want to check you are correct with spelling etc.
* Then make the final migrations by `python3 manage.py migrate`.
\
&nbsp;
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
\
&nbsp;
## Views
* Note: When creating a new view, you need to create the view code, create a template to render the view and then connect up our URLs in the `urls.py` file.

#### Class-based Views:
* Class-based views allow us to make code that's reusable i.e. one view can inherit from another, which is not possible with standard function-based views. That means we can make use of some of the built-in features with Django, such as generic views.
* Each time we create a new view, we need to do three things - Create the view code, create the view template and connect up the urls file.
* Import a generic view by using `from django.views import generic` and create a class which inherits `generic.ListView` - useful if you have a list of objects you want to print to screen.
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
* Open up the `urls.py` file for the overall project to connect the app's urls.
\
&nbsp;
## Project Specific - Viewing the Post's details in a New Page

#### Creating an Associated View
* In `views.py`, import `View` from `django.views` if not using a generic pattern for the view. [More Details](https://stackoverflow.com/questions/48506898/what-are-the-differences-between-generics-views-viewsets-and-mixins-in-django).
* Also import `get_object_or_404` from `django.shortcuts` to check if a post exists.
* Unlike generic views, you have to create functions for the request methods like `get` and `post` - follow the sample for `PostDetail` in `views.py`.
* To check if a user has done a specific action, you can filter using `id=self.request.user.id` within the class.
* Don't forget to return the rendered post with the suitable request, required template name and a library of corresponding post context.

#### Connecting to the suitable URL
* Go to `urls.py` within the app folder and create a new path. As we intend to use the slug in the link, set the first parameter to `<slug:slug>/`, where the first slug is a path converter and the second slug is the `slug` keyword name that matches the `slug` parameter in the associated `PostDetail` class in the `views.py` file within the app folder.
* Note the use of an `<a>` tag in the index file which consists of the post card that connects to the suitable page of the article in full. It has a `href` that looks similar to `{% url 'post_detail' post.slug %}`. The `'post_detail'` refers to the name of the suitable path in the `urls.py` file within the app folder.
\
&nbsp;
## Authentication

#### `django-allauth`
* This Django library can be used to send password and account confirmation emails, enforces password complexity and provides single sign on using Google and Facebook etc.
* Install by `pip3 install django-allauth` and make sure to add to `requirements.txt` with `pip3 freeze --local > requirement.txt`.
* All `allauth` urls need to be added to the project folder's `urls.py` file, as seen in the project.

#### Updating `settings.py`
* Add `ACCOUNT_EMAIL_VERIFICATION = 'none'` to the project's `settings.py` file ton ensure no error `500` can occur during login and registration.
* Add the following to the `INSTALLED_APPS` list:
    * `'django.contrib.sites'` - It's a hook for associating objects and functionality to particular websites, and it's a holding place for the domain names.
    * `'allauth'` - For user authentication.
    * `'allauth.account'` - For dealing with user accounts.
    * `'allauth.socialaccount'` - For dealing with user social media account logins.
* Add `SITE_ID = 1` so that django can handle multiple sites from one database.
* Add `LOGIN_REDIRECT_URL = '/'` and `LOGOUT_REDIRECT_URL = '/'` so that redirects the user to the homepage everytime they login or logout.
* Make sure to `python3 manage.py migrate` any changes.

#### Sign Up
* `allauth` by default then creates a sign up page which can be found by adding `/accounts/signup` to the primary url.
* To connect the html links to these extra pages, use code like `{% url 'account_signup' %}` to sign up and similarly `{% url 'account_logout' %}` for logging out and `{% url 'account_login' %}` for logging in.

#### `allauth` templates
* You firstly need to know what version of Python you are using. Type `ls ../.pip-modules/lib` into the terminal to find out.
* Copy all installed modules into the templates directory by using `cp -r ../.pip-modules/lib/python3.8/site-packages/allauth/templates/* ./templates`, where `-r` refers to recursively, which means to include all directories and the python version you are using can be inserted into the link. The asterisk means all files and then type the directory you want to save all the modules into, which is `templates` in this case.
* Within the `templates` folder, several folders are created, but the one of interest is the `account` folder. This consists of several templates that Django relies on for different tasks.
* For example, within `login.html` it can be changed to match the current template of the other pages i.e. you can remove the `account/` in front of the `base.html` to utilise the existing base file.
\
&nbsp;
## Commenting
* `django-crispy-forms` will be used to create the commenting form and can be downloaded and added to `requirements.txt` as previously mentioned.

#### Integrating the Comments
* In `settings.py`, add `crispy_forms` to the `INSTALLED_APPS` list and add `CRISPY_TEMPLATE_PACK = 'bootstrap4'`, where even though we are using Bootstrap 5, at the time of creating the project, `crispy` only provides support for Bootstrap 3 and 4, so 4 will suffice.
* In order to create the comment form, create a new file called `forms.py` within the app folder and import the `Comment` model as this has all the comment info and the `forms` as the base forms class, which is inherited as `forms.ModelForm`.
* Within the `CommentForm` class, a `Meta` class is created which uses the `Comment` model and the fields are `body` within a tuple. Ensure there is a comma after the word `body` if no other fields are included, otherwise Django will treat it as a string, which will throw an error.
* This comment form can be imported at the top of `views.py` and can be rendered in the post within the `PostDetail` class as `'comment_form': CommentForm()`, where the variable `comment_form` is equal to whatever the imported comments are.

#### Rendering the Comments on the Site
* Make sure to add `{% load crispy_forms_tags %}` to the top of the required page (`post_detail.html` in this example).
* Can show the comments at the end of the page in this example in `post_detail.html`, but note the `| crispy` filter use, which formats the comments nicely.
* Cross Site Request Forgery or a CSRF token is also added, as a CSRF attack can happen when a malicious website contains a link, a form button or some JavaScript that is intended to perform some action on your website, using the credentials of a logged-in user who visits the malicious site in their browser.

#### Obtaining Comment Info from the Site
* Similar to the `get` class method, `post` needs to be created, which will be very similar to `get`.
* A variable `comment_form` (same as the variable in `views.py`) is created and set to the class created within `forms.py`, but the `data` parameter equals `request.POST`, getting all the data posted from the comment.
* Check if all the correct fields have been filled in with the `is_valid()` method. Therefore, instance details can be obtained from the user who has made the post request.
* Call the `.save()` method to save the comment, but set `commit=False` to prevent committing it to the database, as you first want to assign a blog post to it in this example.
* If the comment is not valid, you can set it to a blank element by not letting the data equal to anythin within `CommentForm()`.
* Within the `get` and `post` class methods, a `'commented'` variable is created to let the user know that their comment is waiting approval.
* This `'commented'` variable is used within `post_detail.html` to alert the user of the approval requirement.
\
&nbsp;
## Liking
* Similar to the post detailing and list views, create a class, which checks if a post exists.
* Check if the user has liked the post. If it has then remove the like when they click on the love heart, otherwise add them to the list of users who have liked the post and alter the love heart. 
* In `views.py`, `HttpResponseRedirect` is imported to reload the `post_detail.html` template so you can see the results live without having to refresh the page.
* Also import `reverse`, so you can look up a URL by the name that you give it in the `urls.py` file.
* Return the redirected response, but add the slug as an argument so django knows which page to load.
* Return to `post_detail.html` and add the option to like if the user is authenticated making the icon solid if liked and only an outline if not liked. Make sure to include an `else` block if the user is not authenticated which only shows the love heart icon and the number of likes.
* Finally create the url in the app folder's `urls.py` file which is the similar to the `PostDetail` one but the name is `post_like` to match the reference in the form in `post_detail.html`.