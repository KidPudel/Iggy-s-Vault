Secure Shell

generation
```zsh
ssh-keygen -t rsa -b 4096 -C "email@example.com"
```

to use when running gitbash to clone repo
```zsh
ssh-agent $(ssh-add /Users/youname/.ssh/id_rsa; git clone git@gitlab.com:xyz/SpringBootStarter.git)
```

or add it like that
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519 # or ~/.ssh/id_rsa
```

connect
```zsh
ssh -i ~/.ssh/id_rsa_new root@157.230.112.73
or
ssh kidpudel@65.109.108.151
```

