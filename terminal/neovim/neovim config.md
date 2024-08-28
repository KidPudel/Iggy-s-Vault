to config neovim correctly, we can look at `:h rtp`, to see the run time path, which will tell you where lua looks for configs

the main interesting to as is `<config-path>/nvim` root of Neovim config and `/after` which is used to load configurations of <u>plugins</u> **after** the standard runtime files, so this is basically allows to overrule/override distributed or system-wide settings

---

# basics

the "entry" file is `init.lua`, it takes all configs and requires/loads external settings

`lua/` directory, makes every content inside it **requirable** by the lua, which allows for separation
so if create `nvim/lua/<name>/init.lua`, to make you abstracted configuration and import that inside main `init.lua`
by `require("<name>")`, `nvim/lua/<name>/init.lua` will be called inside `nvim/init.lua`

importing
- `require("directory")`: imports the entry content of the directory (init.lua)
- `require("directory.filename")`: imports only specified file



to remap things use `remap.lua`


# get the plugin manager and fuzzy finder

the go to plugin manager formerly is packager, but now is lazy

[lazy vim setup](https://lazy.folke.io/installation)
after adding lazy to the configuration, we can add plugins by putting them into `my-conf/plugins`, adding them to the `init.lua` and specifying the correct path to the plugin':s `init.lua` 

# configuring plugin:
> NOTE: `config` is executed when plugin is loaded, here you can deeply config some dependencies and plugin itself, by requiring it (so mapping as well), but it is **not** advisable to use it, stick to the `opts`, 
> since default implementation will automatically run `require(MAIN).setup(opts)`, if `opts` or `config` is set to `true`.
> or run `keys`. 
> `keys` are used to define mapping that will trigger/lazily load a plugin and execute a command



mapping is typically happens in this order:
- key map
- command that will be executed by hitting those keys
- sometimes mode
- description
quite simple

```lua
return {
	'nvim-telescope/telescope.nvim', tag = '0.1.8',
	dependencies = { 'nvim-lua/plenary.nvim' },
	keys = {
		{"<leader>pf", "<cmd>Telescope find_files<cr>", desc = "Find files"}
    }
}
```
> ==Always== use `opts` instead of `config` when possible. `config` is almost never needed.

dap?

> Note, some plugins should be forced to load, with priority and lazy to false, and then forced to be applied with `config = function()` require to load



