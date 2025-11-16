fuzzy finder
```zsh
brew install fzf
```

default `fzf` command lets you search for files in a directory and prints it on pick
```
fzf
```

we can also preview
```zsh
fzf --preview "cat {}" # bring to the {} file
```
```zsh
fzf --preview "bat --color=always {}" # bring to the {} file
```


we can also combine it with nvim feature of `nvim <name_of_file>` to open
```zsh
nvim $(fzf --preview "bat --color=always {}") # bring to the {} file
```

this will make us super productive by allowing to search and edit almost at the speed of light
> perfect time for aliasing


# autocomplete
```
cd ~/dotfiles/**
```
will trigger fzf to autocomplete path

```zsh
kill -9 **
```
this will fzf for processes

# key bindings

Firstly to use them, we need to set up shell integration
- bash
    
    ```shell
    # Set up fzf key bindings and fuzzy completion
    eval "$(fzf --bash)"
    ```
    
- zsh
    
    ```shell
    # Set up fzf key bindings and fuzzy completion
    source <(fzf --zsh)
    ```
    
- fish
    
    ```fish
    # Set up fzf key bindings
    fzf --fish | source
    ```


---

- `CTRL-T` - Paste the selected files and directories onto the command-line
- `CTRL-R` - Paste the selected command from history onto the command-line
- `ALT-C` - cd into the selected directory
- `ALT-Q` - add to quick list



# Search syntax

| Token     | Match type                              | Description                                  |
| --------- | --------------------------------------- | -------------------------------------------- |
| `sbtrkt`  | fuzzy-match                             | Items that match `sbtrkt`                    |
| `'wild`   | exact-match (quoted)                    | Items that include `wild`                    |
| `'wild'`  | exact-boundary-match (quoted both ends) | Items that include `wild` at word boundaries |
| `^music`  | prefix-exact-match                      | Items that start with `music`                |
| `.mp3$`   | suffix-exact-match                      | Items that end with `.mp3`                   |
| `!fire`   | inverse-exact-match                     | Items that do not include `fire`             |
| `!^music` | inverse-prefix-exact-match              | Items that do not start with `music`         |
| `!.mp3$`  | inverse-suffix-exact-match              | Items that do not end with `.mp3`            |
|           |                                         |                                              |