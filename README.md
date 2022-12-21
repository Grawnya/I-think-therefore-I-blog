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