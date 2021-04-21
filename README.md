# Django running on Vercel


## Tutorial


### Install Django

```
$ mkdir vercel-django-example
$ cd vercel-django-example
$ pip install Django
$ django-admin startproject vercel_app .
```

### Add an app

```
$ python manage.py startapp example
```

Add the new app to your application settings (`vercel_app/settings.py`):
```python
# vercel_app/settings.py
INSTALLED_APPS = [
    # ...
    'example',
]
```

Be sure to also include your new app URLs in your project URLs file (`vercel_app/urls.py`):
```python
# vercel_app/urls.py
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
            <h1>Hello from Vercel!</h1>
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

Create a new file `vercel.json` and add the code below to it:
```json
{
    "builds": [{
        "src": "vercel_app/wsgi.py",
        "use": "@ardnt/vercel-python-wsgi",
        "config": { "maxLambdaSize": "15mb" }
    }],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "vercel_app/wsgi.py"
        }
    ]
}
```
This configuration sets up a few things:
1. `"src": "vercel_app/wsgi.py"` tells Vercel that `wsgi.py` contains a WSGI application
2. `"use": "@ardnt/vercel-python-wsgi"` tells Now to use the `vercel-python-wsgi` builder (you can
   read more about the builder at https://github.com/ardnt/vercel-python-wsgi)
3. `"config": { "maxLambdaSize": "15mb" }` ups the limit on the size of the code blob passed to
   lambda (Django is pretty beefy)
4. `"routes": [ ... ]` tells Now to redirect all requests (`"src": "/(.*)"`) to our WSGI
   application (`"dest": "vercel_app/wsgi.py"`)


#### Add Django to requirements.txt

The `vercel-python-wsgi` builder will look for a `requirements.txt` file and will
install any dependencies found there, so we need to add one to the project:
```
# requirements.txt
Django==2.2.4
```


#### Update your Django settings

First, update allowed hosts in `settings.py` to include `.now.sh`:
```python
# settings.py
ALLOWED_HOSTS = ['.vercel.app']
```

Second, get rid of your database configuration since many of the libraries Django may attempt to
load are not available on lambda (and will create an error when python can't find the missing
module):
```python
# settings.py
DATABASES = {}
```


### Deploy

With now installed you can deploy your new application:
```
$ vercel
Vercel CLI 21.3.3
? Set up and deploy “vercel-django-example”? [Y/n] y
...
? In which directory is your code located? ./
...
✅  Production: https://vercel-django-example.vercel.app [copied to clipboard] [29s]
```

Check your results in the [Vercel dashboard](https://vercel.com/dashboard).
