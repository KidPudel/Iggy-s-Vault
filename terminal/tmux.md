terminal [[multiplexer]] is a terminal which allows you instead of having one "pseudo" login session, have multiple and ability to detach from it **and reattach**

Benefits:
- Persistent (detach, attach)
- Session sharing (multiple users can use one tmux session)
- Multiple windows/paints ✨


`tmux` to start a new session

# *my* bindings
- `leader-c`: create a **new** window
	- we see current window with opened like this at the bottom `0] 0:zsh* 1:zsh  2:zsh-`
- `leader-n`: move/navigate to next window
	- or `leader-<number of the window>`
- `leader-p`: move to the previous window
- `leader-&`: kill window
- `leader-%`: split window vertical
- `leader-"`: split window horizontally
- `leader-x`: kill pane
- `leader-<hjkl>`: to navigate between panes
- `leader-x`: close pane or `exit`
- `leader-d`: detach from a session
- `tmux ls`: see list of sessions
- `tmux attach`: reattach to the session
- **`leader-s`: manage all session is tmux and allow to choose the session**
- `tmux kill-session -t <number>`: kill session, or `exit`
- `leader-r`: reload tmux
- `leader-I`: install plugins
- `leader-U`: update plugins
- `leader-{}`: switch panes places
- `leader-s`: manage sessions
- `leader-z`: zoom to the particular pane
- `leader-ctrl-z`: zoom to the particular pane
	- `fg` to recover
- `leader-space`: rotate

# vi
- `leader-[`: enter
- `enter`: exit
- `space`/`V`: visual mode
- with yank plugin copy paste as usual cmd-c and cmd-v


# command mode
- `leader-:`: command mode in tmux
- `:rename-window <name>`: rename
- `:rename-session`


# customizations
## status bar
we can change status bar by `set-option -g`
- `status-left`
- `status-right`
- `status-justify`

To add results of bash commands use `#()` syntax 


# commands
`tmux ` -> tab: see the list of commands
`tmux new -s name`: new session with name
`tmux attach -t <name>`


>HINT: select and open links using `shift`



# MArking?
leader + m