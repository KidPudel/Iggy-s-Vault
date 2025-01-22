FIFO is a named pipe is a special file that allows for [[IPC]] and it has no contents on a filesystem
Unlike regular pipes `|` (like in `ls | grep "Error"`), FIFO has name and can be accessed by unrelated processes.

it is like [[channel]]

Allowing:
- data transfer across different processes real-time. No need for [[long polling]]. No intermediate storage, data isn't persistent as file.
- synchronized with mutex
- buffer by kernel, no I/O is happening [[IO-bound or CPU-bound processing]]


```zsh
❯ mkfifo git_aware_pipe
```

```zsh
❯ ls -l git_aware_pipe
prw-r--r--@  1 iggysleepy  staff      0 Jan 22 13:18 git_aware_pipe
```

```zsh
❯ tail -f git_aware_pipe
```

```zsh
❯ echo "hello world" > ~/dev/job/waterfall/repeatable-migrations/git_aware_pipe
```

```zsh
❯ tail -f git_aware_pipe
hello world
```


# command execution using eval ([[terminal-commands]])

```zsh
❯ eval "$(cat git_aware_pipe)"
```
>Note: here we can utilize bash scripting like
```zsh
❯ while true; do eval "$(cat git_aware_pipe)"; done
```


```zsh
❯ echo "ls -l" > ~/dev/job/waterfall/repeatable-migrations/git_aware_pipe
```

```zsh
❯ eval "$(cat git_aware_pipe)"
-rw-r--r--@  1 iggysleepy  staff  87886 Jan 16 22:27 bank_configs.csv
prw-r--r--@  1 iggysleepy  staff      0 Jan 22 13:27 git_aware_pipe
-rw-r--r--@  1 iggysleepy  staff     40 Jan 16 22:05 go.mod
-rw-r--r--@  1 iggysleepy  staff   1013 Jan 17 11:02 main.go
drwxr-xr-x@ 46 iggysleepy  staff   1472 Jan 20 11:40 migrations

```

