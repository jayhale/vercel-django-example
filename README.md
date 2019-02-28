# Django running on Zeit Now


## Tutorial


### Install Django

```
$ mkdir now-django-example
$ cd now-django-example
$ pip install Django
$ django-admin startproject now_app .
```

### Add an app

```
$ python manage.py startapp example
```

Add the new app to your application settings (`now_app/settings.py`):
```python
# settings.py
INSTALLED_APPS = [
    # ...
    'example',
]
```

Be sure to also include your new app URLs in your project URLs file (`now_app/urls.py`):
```python
# now_app/urls.py
from django.urls import path, include

urlpatterns = [
    ...
    path('', include('example.urls')),
]
```


#### Create the first view

Add the code below (a simple view that returns the current time) to `example/views.py`:
```python
# example/views.py
from datetime import datetime

from django.http import HttpResponse


def index(request):
    now = datetime.now()
    html = f'''
    <html>
        <body>
            <h1>Hello from Zeit Now!</h1>
            <p>The current time is { now }.</p>
        </body>
    </html>
    '''
    return HttpResponse(html)
```


#### Add the first URL

Add the code below to a new file `example/urls.py`:
```python
# example/urls.py
from django.urls import path

from example.views import index


urlpatterns = [
    path('', index),
]
```


### Test your progress

Start a test server and navigate to `localhost:8000`, you should see the index view you just
created:
```
$ python manage.py runserver
```

### Get ready for Now

#### Add the Now configuration file

Create a new file `index.py` to act as your application entrypoint for Now. The file should import
the WSGI application you want to use and make it available as `application`:
```python
# index.py
from now_app.wsgi import application
```

Create a new file `now.json` and add the code below to it:
```json
{
    "version": 2,
    "name": "python-wsgi-example",
    "builds": [{
        "src": "index.py",
        "use": "@ardent-labs/now-python-wsgi",
        "config": { "maxLambdaSize": "15mb" }
    }]
}
```
This configuration sets up a few things:
1. `"src": "index.py"` tells Now that `index.py` is the only application entrypoint to build
2. `"use": "@ardent-labs/now-python-wsgi"` tells Now to use the `now-python-wsgi` builder (you can
   read more about the builder at https://github.com/ardent-co/now-python-wsgi)
3. `"config": { "maxLambdaSize": "15mb" }` ups the limit on the size of the code blob passed to
   lambda (Django is pretty beefy)


#### Update your Django settings

First, update allowed hosts in `settings.py` to include `now.sh`:
```python
# settings.py
ALLOWED_HOSTS = ['*.now.sh']
```

Second, get rid of your database configuration since many of the libraries django may attempt to
load are not available on lambda (and will create an error when python can't find the missing
module):
```python
# settings.py
DATABASES = {}
```


### Deploy

With now installed you can deploy your new application:
```
$ now
> Deploying now-django-example under jayhale
...
> Success! Deployment ready [57s]
```

Check your results by visiting https://zeit.co/dashboard/project/python-wsgi-example
