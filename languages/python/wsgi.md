Web Server Gateway Interface, standard for interaction between Python and backend server itself, basically to ***forward requests from web server to the python*** programming applications.

roughly saying, it's like an `input()` that is waiting for the [[nginx]] to format/convert the request to the acceptable for wsgi form and pass to pass it to the python program

The most popular realization for wsgi in python is [[gunicorn]]