Simple Python Version Manager.
Lets you easily switch between multiple python versions

```zsh
brew install pyenv
```

add [[environment variables]] `PYENV_ROOT` and then add it to the $PATH
```zsh
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
```

and then run `eval "$(pyenv init -)"` to install pyenv into your sheel as shell function and enable shims and autocompletion

or just run in `.zshrc`

```zsh
export PYENV_ROOT="$HOME/.pyenv"
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
eval "$(pyenv init -)"
```

and now to install and switch versions
```
pyenv install 3.10.14
pyenv global 3.10.14
```

and to list all versions
```
pyenv versions
* system (set by /Users/iggysleepy/.pyenv/version)
  3.10.14
  3.12.4
```

and to see your version you've switched to 3.10.14

```
$ pyenv version
3.10.14 (set by /Users/iggysleepy/.pyenv/version)

$ python -V
Python 3.10.14
```

> NOTE: to return to the system version `pyenv global system`