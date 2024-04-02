To add environment variable we can run `export VARIABLENAME="PATH"` directly in a terminal, but this will only set a session scoped variable, 
so instead of it, we need to add a new variable to the shell configuration of our terminal `~/.zshrc`
```bash
export PATH="$PATH:$(command to get the path)/bin"
```