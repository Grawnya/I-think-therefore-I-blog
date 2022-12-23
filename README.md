# I think, Therefore I Blog Django Project

#### Set Up
* Note: Closely follow the [Django Notes](https://github.com/Grawnya/django-notes) repo, as it outlines how to start up the project and provides more primary info on Django & it's nuances.
* Commands:
    - `pip3 install 'django<4' gunicorn`
    - `pip3 install dj_database_url==0.5.0 psycopg2`
    - `pip3 install dj3-cloudinary-storage`
    - `pip3 freeze --local > requirements.txt`
    - `django-admin startproject codestar .`
    - `python3 manage.py startapp app_name`
* Then add the app name to the `settings.py` file in the `INSTALLED_APPS` list.
* The migration:
    - `python3 manage.py migrate`
    - `python3 manage.py runserver` (to check site works)
* Heroku & ElephantSQL DB:
    - Create a Heroku App
    - Create an ElephantSQL DB and copy the URL
    - Create an `env.py` file and save both the Django secret key and DB URL values
    - In `settings.py`, set the `SECRET_KEY` variable to the saved one in `env.py`.
    - Also comment out the `DATABASE_URL` variable, creating a copy of one, equal to a dictionary where `default : dj_database_url.parse(os.environ.get('DATABASE_URL'))`
    - `python3 manage.py migrate` (to migrate your database structure to the newly-connected ElephantSQL DB)
    - Add both the `DATABASE_URL` and `SECRET_KEY` to your Heroku project's config vars, as well as `PORT` is 8000.
* Cloudinary:
    - Create an account and copy the URL from your dashboard.
    - Create a variable called `CLOUDINARY_URL` and set it to the URL in both the `env.py` file and the Heroku config vars.
    - Also create a config var called `DISABLE_COLLECTSTATIC` and set it to 1, to ensure the deployment does not fail - but remove it when deploying the final project at end.
    - In `settings.py`, in the `INSTALLED APPS` list, add `cloudinary_storage` above `django.contrib.staticfiles` and put `cloudinary` just above any self created apps.
    - Under `STATIC_URL` add `STATICFILES_STORAGE` equal to `Cloudinary_storage.storage.StaticHashedCloudinaryStorage`, which connects to the `cloudinary_storage` app.
    - Add also `STATICFILES_DIRS` and `STATIC_ROOT`, which are the locations of where the files will be stored and the suitable root, but the latter will not be used in this project.
    - Similarly, create the variables `MEDIA_URL` and `DEFAULT_FILE_STORAGE` which is specifically set to `cloudinary_storage.storage.MediaCloudinaryStorage`.
    - Add a `TEMPLATES_DIR` variable under the `BASE_DIR` to connect to the templates folder and within the `TEMPLATES` variable, add the `TEMPLATES_DIR` value to the list of the `APP_DIRS` value.
    - Before deployment, set the `ALLOWED_HOSTS` variable to `the_name_of_heroku_app.herokuapp.com` and add `localhost` to the list so you can still work on the project locally. 
* Final Steps:
    - Create `static`, `templates` and `media` folders at the top level as mentioned in the `settings.py` file.
    - Create a Procfile with the command `wed: gunicorn codestar.wsgi` and commit the project to ensure it is up-to-date.

