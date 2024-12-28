> Command-line commands used to enter Ex commands (":"), search patterns ("/" and "?"), and filter commands ("!")
# Ex Commands
- `:h {topic}` or `:help`: help (optionally, the topic) **MUST HAVE**
- `:help user-manual`: open Neovim user manual in a browser
- `:w {file}`: save (optionally file)
- `:wa`: save all
- `:q`: exit
- `:wq`: save and quit
- `:q!`: quit without saving
- `:ls`: list open buffers
- `:split`: split window horizontally
	- `:vsplit`: vertically
- `:tabnew`: open a new tab
- `:tabclose`: close the current tab
- `:{tab number}gt`: go to the tab number
- `:e {file}`: open the file
	- `:e#`: move to the previous file
- `:b {buffer number}`: switch to the specified buffer
- `:bd`: close current buffer
- `:syntax on/off`: enable/disable syntax highlighting
- `:checkhealth <>`: check health of a 
- `so`: source to apply changes without relaunching
- `:Ex`: open [[vim netrw mode]]
- `:lua` to execute lua code
- `:messages`: to see the log of messages

# macros
> macro is the list of keypresses, that you could store in a buffer and repeat it

we can register them by typing
- `:new`: to create a new macro  or the next step right away
	- `q<letter of the register to put macro into>`: denotes you want to create a macro now (also this will end the macro recording)
- `:put <letter>`: put the content of the register
- `:reg`: to list all registers

## quick macro
1. `q` for record -> `any letter to assign macro`
2. record macro
3. end with hitting `choosen letter`
4. `@<letter>`: to replay it!
> very powerful thing


# Search patterns
`/`: start searching
cool little example:
find containing `vim.`
```zsh
/vim.
```
find 
`*`: find/highlight all matching words under the cursor 



# Replace
```
:%s/original/replacement/g(for global)c(ask)
```
where `.` could mean any character