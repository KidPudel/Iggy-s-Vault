terminal [[multiplexer]] is a terminal which allows you instead of having one "pseudo" login session, have multiple and ability to detach from it **and reattach**

Benefits:
- Persistent (detach, attach)
- Session sharing (multiple users can use one tmux session)
- Multiple windows/paints ✨


`tmux` to start a new session

# *my* commands (.tmux.conf)
- `leader-c`: create a **new** window
	- we see current window with opened like this at the bottom `0] 0:zsh* 1:zsh  2:zsh-`
- `leader-n`: move/navigate to next window
	- or `leader-<number of the window>`
- `leader-&`: kill window
- `leader-%`: split window vertical
- `leader-"`: split window horizontally
- `leader-x`: kill pane
- `leader-<hjkl>`: to navigate between panes
- `leader-x`: close pane or `exit`
- `leader-d`: detach from a session
- `tmux ls`: see list of sessions
- `tmux attach`: reattach to the session
- `leader-s`: list all session is tmux and allow to choose the session
- `tmux kill-session -t <number>`: kill session, or `exit`
- `leader-r`: reload tmux
- `leader-I`: install plugins


# command mode
- `leader-:`: command mode in tmux
- `:rename-window <name>`: rename
- `:rename-session`


