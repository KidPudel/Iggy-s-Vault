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
    - The list is generated using `--walker file,dir,follow,hidden` option
        - You can override the behavior by setting `FZF_CTRL_T_COMMAND` to a custom command that generates the desired list
        - Or you can set `--walker*` options in `FZF_CTRL_T_OPTS`
    - Set `FZF_CTRL_T_OPTS` to pass additional options to fzf
        
        ```shell
        # Preview file content using bat (https://github.com/sharkdp/bat)
        export FZF_CTRL_T_OPTS="
          --walker-skip .git,node_modules,target
          --preview 'bat -n --color=always {}'
          --bind 'ctrl-/:change-preview-window(down|hidden|)'"
        ```
        
    - Can be disabled by setting `FZF_CTRL_T_COMMAND` to an empty string when sourcing the script
- `CTRL-R` - Paste the selected command from history onto the command-line
    - If you want to see the commands in chronological order, press `CTRL-R` again which toggles sorting by relevance
    - Press `CTRL-/` or `ALT-/` to toggle line wrapping
    - Set `FZF_CTRL_R_OPTS` to pass additional options to fzf
        
        ```shell
        # CTRL-Y to copy the command into clipboard using pbcopy
        export FZF_CTRL_R_OPTS="
          --bind 'ctrl-y:execute-silent(echo -n {2..} | pbcopy)+abort'
          --color header:italic
          --header 'Press CTRL-Y to copy command into clipboard'"
        ```
        
- `ALT-C` - cd into the selected directory
    - The list is generated using `--walker dir,follow,hidden` option
    - Set `FZF_ALT_C_COMMAND` to override the default command
        - Or you can set `--walker-*` options in `FZF_ALT_C_OPTS`
    - Set `FZF_ALT_C_OPTS` to pass additional options to fzf
        
        ```shell
        # Print tree structure in the preview window
        export FZF_ALT_C_OPTS="
          --walker-skip .git,node_modules,target
          --preview 'tree -C {}'"
        ```
        
    - Can be disabled by setting `FZF_ALT_C_COMMAND` to an empty string when sourcing the script