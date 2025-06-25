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

NOTE: it is only for session life, add it to the zshrc to make persistent

also check in config

```
# BWG
Host gitlab.aaaaaa.abc
  HostName gitlab.aaaaaa.abc
  User git
  PreferredAuthentications publickey
  IdentityFile ~/.ssh/id_rsa_aaa
```

also verify known_hosts which contains public keys of remote hosts you've connected to, it could be outdated, but it will prompt you if so

connect
```zsh
ssh -i ~/.ssh/id_rsa_new root@000.000.000
or
ssh kidpudel@00.000.0000.00000
```

