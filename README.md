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

Create a new file `now.json` and add the code below to it:
```json
{
    "version": 2,
    "name": "now-django-example",
    "builds": [{
        "src": "now_app/wsgi.py",
        "use": "@ardnt/now-python-wsgi",
        "config": { "maxLambdaSize": "15mb" }
    }],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "now_app/wsgi.py"
        }
    ]
}
```
This configuration sets up a few things:
1. `"src": "now_app/wsgi.py"` tells Now that `wsgi.py` contains a WSGI application
2. `"use": "@ardnt/now-python-wsgi"` tells Now to use the `now-python-wsgi` builder (you can
   read more about the builder at https://github.com/ardnt/now-python-wsgi)
3. `"config": { "maxLambdaSize": "15mb" }` ups the limit on the size of the code blob passed to
   lambda (Django is pretty beefy)
4. `"routes": [ ... ]` tells Now to redirect all requests (`"src": "/(.*)"`) to our WSGI
   application (`"dest": "now_app/wsgi.py"`)


#### Add Django to requirements.txt

The `now-python-wsgi` builder will look for a `requirements.txt` file and will
install any dependencies found there, so we need to add one to the project:
```
# requirements.txt
Django==2.2.4
```


#### Update your Django settings

First, update allowed hosts in `settings.py` to include `.now.sh`:
```python
# settings.py
ALLOWED_HOSTS = ['.now.sh']
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

Check your results by visiting https://zeit.co/dashboard/project/now-django-example
