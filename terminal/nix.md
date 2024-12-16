[[declarative]] package manager
```zsh
$ sh <(curl -L https://nixos.org/nix/install)
```

create config
```zsh
mkdir -p ~/.config/nix
cd ~/.config/nix
```

init flake
```zsh
nix flake init -t nix-darwin --experimental-features 'nix-command flakes'
```

rename 
```zsh
sed -i '' "s/simple/$(scutil --get LocalHostName)/" flake.nix
```

install nix-darwin with flakes
```zsh
nix run nix-darwin --experimental-features 'nix-command flakes' -- switch --flake ~/.config/nix
```

apply changes
```zsh
darwin-rebuild switch --flake ~/.config/nix --impure
```


# Home-manager
home-manager is a user scoped config manager
Just add `home.nix` to the `flake.nix`
but now i use [[stow]]