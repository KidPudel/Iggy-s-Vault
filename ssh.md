Secure Shell

generation
```zsh
ssh-keygen -t rsa -b 4096 -C "email@example.com"
```

some use
```zsh
ssh-agent $(ssh-add /Users/iggysleepy/.ssh/id_rsa; git clone git@git.bwg-io.site:waterfall/payment-order-service.git)
``` 