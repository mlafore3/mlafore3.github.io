---
layout: project
title:  "Build-a-Business - a SaaS application template"
date:   2018-03-13
excerpt: ""
tag:
- project
---
I am always thinking of new ideas for web applications, so is my brother so maybe it's in our DNA. There is a problem however, just getting a project up and running takes a significant amount of time. A SaaS application has a lot of moving parts. At minimum, you need to setup a database, a backend API and a frontend. This problem came up time and time again so I decided to develop a base SaaS application project that I could use to jumpstart any of my ideas. The chosen stack is a postgresql database, django-rest backend API and a frontend powered by react and redux. I utilized djangos and reacts boiler plate project templates. The django one is pretty good but I heavily changed the react template. If you're only intered in the code, the repository can be found on [github](https://github.com/mlafore3/build-a-business).

Here are the steps that I followed to build the app architecture

{% highlight sh %}
$ mkdir django_react_template
$ virtualenv env
$ source env/bin/activate
$ pip3 install django djangorestframework django-filter
$ pip3 freeze > requirements.txt

$ django-admin startproject backend
$ cd backend
$ django-admin startapp api
{% endhighlight %}

https://www.postgresql.org/files/documentation/pdf/9.6/postgresql-9.6-A4.pdf

http://www.marinamele.com/taskbuster-django-tutorial/install-and-configure-posgresql-for-django

{% highlight sh %}
$ createdb django_react_template_db
$ psql django_react_template_db
{% endhighlight %}

{% highlight sql %}
# CREATE ROLE django_react_template_user WITH LOGIN PASSWORD 'T3stpa55w0rd';
# GRANT ALL PRIVILEGES ON DATABASE django_react_template_db TO django_react_template_user;
# ALTER USER django_react_template_user CREATEDB;
{% endhighlight %}

Some useful commands are 
{% highlight sh %}
$ dropdb mydb
psql --username=postgres
\l # list databases
\q # exit 
postgres=# drop database mydb;
{% endhighlight %}

http://www.marinamele.com/taskbuster-django-tutorial/settings-different-environments-version-control

{% highlight sh %}
$ cd backend
$ mkdir settings
$ touch development.py production.py staging.py testing.py
{% endhighlight %}

{% highlight sh %}
# move the settings.py to this folder and change its name to base.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': get_env_variable('DATABASE_NAME'),
        'USER': get_env_variable('DATABASE_USER'),
        'PASSWORD': get_env_variable('DATABASE_PASSWORD'),
        'HOST': '',
        'PORT': '',
    }
}
{% endhighlight %}

{% highlight sh %} $ cd ~/.virtualenvs/env/bin {% endhighlight %}

{% highlight sh %} $ nano postactivate

export DJANGO_SETTINGS_MODULE="django_react_template.settings.settings.development"
export DATABASE_NAME='django_react_template_db'
export DATABASE_USER='mlaforet'
export DATABASE_PASSWORD='T3stpa55w0rd'
export SECRET_KEY="your_secret_django_key" # grab the secret key from base.py then delete it from the file
cntrl + O
enter
cntrl + X
{% endhighlight %}

{% highlight sh %} $ nano predeactivate

unset DJANGO_SETTINGS_MODULE
unset DATABASE_NAME
unset DATABASE_USER
unset DATABASE_PASSWORD
unset SECRET_KEY
cntrl + O
enter
cntrl + X
{% endhighlight %}

deactivate and reactivate your virtual environment
{% highlight sh %}
python manage.py check
python manage.py migrate
python manage.py createsuperuser
{% endhighlight %}

username: mlaforet
email: ''
password: changeme

http://www.django-rest-framework.org/#installation
http://www.django-rest-framework.org/api-guide/authentication/#tokenauthentication

{% highlight sh %}
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
    ]
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 50,
    'DEFAULT_AUTHENTICATION_CLASSES': ( 
    'rest_framework.authentication.BasicAuthentication',
    'rest_framework.authentication.TokenAuthentication',
    ),
}
{% endhighlight %}

{% highlight sh %}
Go into root of folder
$ npm install -g create-react-app
$ create-react-app frontend
$ cd frontend
$ npm run eject
{% endhighlight %}

{% highlight sh %}
$ npm install --save-dev babel-preset-es2015 babel-preset-stage-3
$ npm install --save redux redux-logger redux-persist react-redux
$ npm install --save axios react-router-dom lodash
{% endhighlight %}

This is where I decided to change the base frontend architecture. To be continued. 

{% highlight sh %}
$ touch store.js
$ mkdir reducers
$ mkdir actions
$ touch reducers/index.js
{% endhighlight %}

{% highlight sh %} npm start {% endhighlight %} in frontend root
