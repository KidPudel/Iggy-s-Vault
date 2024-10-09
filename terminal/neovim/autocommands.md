commands that run automatically on certain events

with the following pattern
```lua
autocmd [group] event pattern command/callback
```

```lua
vim.api.nvim_create_augroup("autosave", {clear = true})
vim.api.nvim_create_autocmd({ "BufLeave", "FocusLost", "VimLeavePre" }, {
	group = "autosave",
	callback = function(table)
	    if vim.api.nvim_buf_get_option(table.buf, "modified") then
            print("save")
            vim.schedule(function()
                vim.api.nvim_buf_call(table.buf, function()
                    vim.cmd "silent! write"
                end)
            end)
        end
	end
})
```

`:h events`: list of events
`:autocmd`: available auto commands