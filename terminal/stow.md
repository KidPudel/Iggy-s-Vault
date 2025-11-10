[[Symlink]] farm manager which takes data, located in separate directories , and makes them appear to be installed in the same place

# Naming conventions
- If we want to symlink `~/.config/nvim`, we need to put it into `./nvim/.config/nvim`
- `~/.config/wezterm` -> `./wezterm/.config/wezterm`
- `~/.zshrc` -> `./zshrc/.zshrc`

```sh
stow nvim
```
this would create a symlink in `~/.config/nvim` directory that is pointing to the `/nvim` directory.
This way we are actually storing our configs in our folder and we creating *links* to our configs, that are used by the systems to apply changes.