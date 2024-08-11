Secure Shell

generation
```zsh
ssh-keygen -t rsa -b 4096 -C "email@example.com"
```

some use
```zsh
ssh-agent $(ssh-add /Users/iggysleepy/.ssh/id_rsa; git clone git@git.bwg-io.site:waterfall/payment-order-service.git``` 
```

connect
```zsh
ssh -i ~/.ssh/id_rsa_new root@157.230.112.73
or
ssh kidpudel@65.109.108.151
```

