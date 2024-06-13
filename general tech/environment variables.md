To add environment variable we can run `export` directly in a terminal, but this will only set a session scoped variable, 
```zsh
export VARIABLENAME="PATH"
```


so instead of it, we need to add a new variable to the shell configuration of our terminal `~/.zshrc`
```zsh
export PATH="$PATH:$(command to get the path)/bin"
```