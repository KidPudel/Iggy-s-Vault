virtual environment is the additional isolated python environment with the python itself installed with specific version as well as interpreter and external python packages, that can be installed independently from system-wide environment as well as with different versions (which allows to separate coupling and dependency on each other)

So you don't need for system-wide installations


Typically virtual environments will use the same Python interpreter it was created with, still tools like [[pyenv]] can be used along side with venv

# setup the project with the venv

Creating a venv for the project
```zsh
python[version] -m[to run library module as script] venv [name for the folder, like venv]
```

Activate venv, to prepare current terminal session to use isolated python environment within virtual environment directory/package
```zsh
myenv\Scripts\activate for windows
source myenv/bin/activate for Unix
```

Now we can install packages for our venv

and then to deactivate
```zsh
myenv\Scripts\deactivate for windows
deactivate for unix
```