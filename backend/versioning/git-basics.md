# [learn go in 15 minutes](https://www.youtube.com/watch?v=USjZcfj8yxE&t=712s)
# basics
- `git init`:  To start tracking your project in git
- `git status`:  shows current status of our repository
- `git log`: shows history of commits
- `git add`:  add our changes to staging area, to be committed
	- `git add .`
	- `git add main.go dockerfile README.md`
- `git reset`: discard changes in specified filed in working directory to a specific commit
	- `git reset HEAD`
	- `git reset --hard <branch-name>`
- `git commit`: a **snapshot** of git repository at a particular *time*
	- `git commit -m ""`: where `-m` is a message
- `git checkout <commit-hash>`: to navigate between your commits and branches (restores it)
	- `git checkout master`: to move to the most recent commit
	- `git checkout -b <branch-name> <for-example-origin>/<from-what-branch>`: checkout to the branch with creation of it
- `git switch`: switches between branches
- `git restore`: restores a file from another source 
- `git stash`: stash changes to the dirty working directory
	- `git stash drop`: drops that changes
	- `git stash pop`: pops changes from dirty to the current branch


# branch
In git's branch could be interpreted as an individual timeline of our git repository.
And by default, git creates a main branch called "*master*"
- `git branch`: lists our branches
	- `git branch -r`: lists remote branches
- `git branch <name>`: creates a new branch with name
> Side note: If we use checkout, we are actually detaching from our master, and to attach back to it we can just use `git checkout master`
- `git merge <branch-name>`: to *rebase* (change the base/branch of the history) changes (commits) from specified branch to the current branch, with a new "merge commit"
	- ![[Pasted image 20240711180045.png]]
- `git rebase <branch-name>`: moves entire current branch to the tip of the chosen branch, and instead of using merge commits, it **re-writes the project history**, by creating **brand new commits for *each commit in the original branch***
	- ![[Pasted image 20240711180949.png]]
- `git delete <branch-name>`: delete branch

- `git branch -m <new-name>`: rename branch (if not pointing, then specify old name as well) and to rename remote, just push this branch

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
- `git remote add <alias> <https://gitub.com/KidPudel/goproject.git>`
- `git remote add upstream <https://github.com/<username>/<projectname>.git`> to add upstream to your configured remote, to pull changes from upstream repository
- `git pull --set-upstream origin main`
- `git pull --allow-unrelated-hisotries`
зайти как не зарегистрированный пользователь


`git remote set-url <existing alias> <url>`: change url to the existing remote


# rebasing on pull
we can do merging on pull instead of rebase, if we run `git config pull.rebase false`


# cherry-pick
`git cherry-pick <commit-hash>`
Used to select a single commit from one branch and apply it to another

Common use cases:

- Applying a bug fix from one branch to another
- Moving a feature commit to a different branch
- Selectively applying changes when you don't want to merge entire branches

# git rebase
видел, что в одном PR ты добавил изменения из мастера в свою ветку через мердж. Это ОК, но лучше было бы сделать это через `git rebase`.

Алгоритм такой:

- `git checkout master`
- `git pull origin master`
- `git checkout <out-branch>`
- `git rebase master`

Данное действие отменит все коммиты в текущей ветке и затем переставит их так, словно ветка началась от текущего состояния мастера. И будет применять коммит за коммитом, так что если будут конфликты, оно остановит процесс и позволит их решить. После чего нужно будет сделать `git rebase --continue`.

Плюс данного подхода в том, что так в PR сохраняется линейная история без непонятных коммитов в виде `Merge ... to ...`. Учитывая, что мы потом не делаем squash (объединение коммитов в один - я против такого, т.к. теряется история), то при ребейзе потом и мастер будет чище и линейнее после вливания PR.

Однако данный подход может быть весьма сложным, если раньше с ним не было опыта. Потому я и не настаиваю на нём. Просто предлагаю обратить внимание.



# git drop changes
- `git stash push -m "Stash .idea changes"`
- `git checkout <destination>`
- `git stash drop`


# git stash

# conflicts
when on both branches altered the same file