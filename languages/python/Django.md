# Django vs [[FastAPI]]
They are serve different niches.
FastAPI shines: projects that leans towards API-centric tasks, also its:
- Performant
- Concurrent
- Minimal
Django: For full-stack applications and not just for [[API]], API is just a part of it, it battery included
- Security
- Authentication
- [[MVT]] approach
- Predefined and Generic Classes

FastAPI for building fast and scalable API, that supports various data formats and real-time communication.
Django for building full-fledged web applications, that supports various web features and functionalities. basically to avoid micro-services

# Get started
To generate the **whole app** with necessary files run
```zsh
django-admin startproject myproject
```


## [[ASGI]] and [[WSGI]] 
files that represent [[application server]] that allows us application to communicate with the [[web server]] 
Those files are used for deploying and for communication with application server to connect with django or other python framework that implements [[WSGI]] specified by python community like [[Gunicorn]] (most of the time [[FastAPI]])
## settings
`settings.py` file that contains settings of application, like database, middleware, templates, authentication, etc

## urils
to configure different url routes
## manage file
`manage.py` acts like a command-line tool that allows us to ***interact with Django app*** like making a database migrations, run application server.
Here we defining a setting module with all configurations, this will tell it which settings you are using
```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'project.settings')
```
or use `configure()` method, one of them is required.
```python
from django.conf import settings

settings.configure(default_settings=myapp_defaults, DEBUG=True)
```

And if we use Django component(s) as a standalone, we **must** call `setup()` to load settings and populate Django's application registry (manage your resources).
But it is necessary only if it is truly standalone. When inked by your web server or through django-admin, Django will handle this for you.
# Django ORM
Django [[ORM]] is for interacting with database using models representing tables 
### Migrations
After creating [[ORM]]s or changing it we need to make a migration, to affect changes to the models into the database schema
Migration is automated code that create a model in some database engine like PostgreSQL

Firs we need to register them in admin

```python
from django.contrib import admin
from .django_models import Client, ClientInfo, Endpoint, EndpointStates

# Register your models here.
admin.site.register([Client, ClientInfo, Endpoint, EndpointStates])

```

```zsh
python manage.py makemigrations # prepares the nexessary SQL commands in a migration file
python manage.py migrate # actually apply changes 
```


The `manage.py` is the command-line utility that lets you interact ***with Django project*** in various ways.
and we can create it by creating a new project

and now to setup a database connection we need to set it in `settings.py`


the example of model [[ORM]]
```python
class Client(models.Model):
	client_name = models.CharField(max_length=256, null=False, blank=False)
	client_info = models.ForeignKey(ClientInfo, on_delete=models.PROJECT, null=True, blank=True)
	class Meta:
		managed = False
		db_table="clients"
		app_label="fast_api"
```


and usage

```python
endpoint_states = (
        EndpointStates.objects.annotate(
            id_mod_3=ExpressionWrapper(F("id") % 3, output_field=IntegerField())
        )
        .filter(Q(endpoint_id=139) & Q(id_mod_3=0) & Q(state_start__gte=seconds))
        .order_by("-state_start")
        .values()
    )
```
# Application
To manage our components, we need application
```python
from django.apps import AppConfig


class FastApiConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'fast_api'

```

