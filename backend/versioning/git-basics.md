# [learn go in 15 minutes](https://www.youtube.com/watch?v=USjZcfj8yxE&t=712s)
# basics
- `git init`:  To start tracking your project in git
- `git status`:  shows current status of our repository
- `git log`: shows history of commits
- `git add`:  add our changes to staging area, to be committed
	- `git add .`
	- `git add main.go dockerfile README.md`
- `git restore`: discard changes in specified filed in working directory 
- `git commit`: a **snapshot** of git repository at a particular *time*
	- `git commit -m ""`: where `-m` is a message
- `git checkout <commit-hash>`: to navigate between your commits and branches
	- `git checkout master`: to move to the most recent commit


# branch
In git's branch could be interpreted as an individual timeline of our git repository.
And by default, git creates a main branch called "*master*"
- `git branch`: lists our branches
- `git branch <name>`: creates a new branch with name
> Side note: If we use checkout, we are actually detaching from our master, and to attach back to it we can just use `git checkout master`
- - `git merge <branch-name>`: to add changes (commits) from specified branch to the current branch
- `git delete <branch-name>`: delete branch


# git setup (login)
```
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
```



# remote

## origin vs upstream
- "origin" is typically the default name for the remote repository main branch, that you've cloned or initially setup as the main remote for local repository. (**default remote repository, that you are directly accessing**,  remote repository you are pushing to)
- upstream is the remote repository that is not the original repository (*you are forked **from***), but that we've cloned from. It is often used when we want to keep track of changes made on the original repository or want to contribute changes back to the original repository. (**additional remote repository you may want to track or contribute to**, something to points to where it originated from)

## -u vs upstream

- `-u`: in context of pushing `git push -u <remote-name> <branch-name>` is the flag used when pushing to remote repository, used to set up tracking relationship for a local branch with a corresponding/specific remote branch, so that your local branch know which remote branch it is associated with and specified remote branch becomes the. default one for local branch, so in the future, when executing commands `git pull` and `git push`, without specifying the remote branch, will default to the upstreamed (tracked) remote branch.
- upstream: in context of setting up a remote, you add a remote repository that is typically forked that is by convention named an "upstream"

- `git remote -v` to see configured remote of repo
- `git remote add origin <https://gitub.com/KidPudel/goproject.git>`
- `git remote add upstream <https://github.com/<username>/<projectname>.git`> to add upstream to your configured remote, to pull changes from upstream repository
- `git pull --set-upstream origin main`
- `git pull --allow-unrelated-hisotries`
зайти как не зарегистрированный пользователь


# rebasing on pull
we can do merging on pull instead of rebase, if we run `git config pull.rebase false`