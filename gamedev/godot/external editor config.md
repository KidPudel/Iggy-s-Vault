for VSCode
exec path: `/Applications/Visual Studio Code.app/Contents/MacOS/Electron`
exec flag: 
```sh
{project} --goto {file}:{line}:{col}
```

---

for Zed
exec path: /Applications/Zed.app/Contents/MacOS/cli
exec flag: {project} {file}:{line}:{col}

---

for neovim
exec path: `nvim`

exec flag:
```sh
--server ./godothost --remote-send "<C-\><C-N>:n {file}<CR>{line}G{col}|"
```

`--server {project}/server.pipe --remote-send "<C-\><C-N>:e {file}<CR>:call cursor({line}+1,{col})<CR>"
127.0.0.1:6004

`--server 127.0.0.1:6004 --remote-send "<C-\><C-N>:e {file}<CR>:call cursor({line}+1,{col})<CR>"`

defaults
```lua
local port = os.getenv 'GDScript_Port' or '6005'
local cmd = vim.lsp.rpc.connect('127.0.0.1', tonumber(port))

---@type vim.lsp.Config
return {
  cmd = cmd,
  filetypes = { 'gd', 'gdscript', 'gdscript3' },
  root_markers = { 'project.godot', '.git' },
}
```



