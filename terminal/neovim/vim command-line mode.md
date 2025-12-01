> Command-line commands used to enter Ex commands (":"), search patterns ("/" and "?"), and filter commands ("!"), to execute commands use :lua command
# Ex Commands
- `:h {topic}` or `:help`: help (optionally, the topic) **MUST HAVE**
- `:help user-manual`: open Neovim user manual in a browser
- `:w {file}`: save (optionally file)
- `:wa`: save all
- `:q`: exit
- `:wq`: save and quit
- `:q!`: quit without saving
- `:s`: replace in a current line
	- `:%s`: The `%` makes it search the entire buffer instead of just the current line
- `:ls`: list open buffers
- `:split`: split window horizontally
	- `:vsplit`: vertically
- `:tabnew`: open a new tab
- `:tabclose`: close the current tab
- `:{tab number}gt`: go to the tab number
- `edit/e`: force reload of a file
	- `e!`: override current
	- `:e#`: move to the previous file
	- `:e {file}`: open the file
- `:b {buffer number}`: switch to the specified buffer
- `:bd`: close current buffer
- `:syntax on/off`: enable/disable syntax highlighting
- `:checkhealth <>`: check health of a 
- `so`: source to apply changes without relaunching
- `:Ex`: open [[vim netrw mode]]
- `:lua` to execute lua code
- `:messages`: to see the log of messages
- `:hi`: highlights
- `:cdo`: execute command on each line in the quickfix list. for specific matching line
	- Example: `:cdo s/models/platform/gc | update`
- `:cfdo`: execute command on each file in the quickfix list. for entire file
- `:cnext`: move to the next item in quickfix list
- `:norm`: Normal mode commands. It will execute command like it was typed out. for example you can select the area and do the thing
- `grep`: [[grep]]
- `vimgrep`: [[grep]] that supports vim regex and adds to a quickfix list

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
basic syntax
```sh
:[range]s/{pattern}/{string}/[flags]
```

## Components Explained:

- `[range]`: Specifies the lines on which the substitution will occur.
    - `:`: If omitted, the substitution applies only to the current line.
    - `%`: Applies to the entire file.
    - `start_line,end_line`: Applies from `start_line` to `end_line`. For example, `1,10` replaces in lines 1 through 10.
    - `.,+3`: Applies from the current line (`.`) to three lines after the current line.
- `s`: The substitute command itself.
- `{pattern}`: The text or regular expression to search for.
- `{string}`: The text to replace the `pattern` with.
- `[flags]`: Optional modifiers for the substitution.
    - `g`: (Global) Replaces all occurrences of `pattern` on each line within the specified range, not just the first one.
    - `c`: (Confirm) Prompts for confirmation before each replacement.
    - `i`: (Case insensitive) Performs a case-insensitive search.
    - `I`: (Case sensitive) Performs a case-sensitive search (overrides `i` if set in options).

## Basic Substitution with Capture Groups:

- **The Command:** The general format is `:s/pattern/replacement/flags`.
- **Defining Capture Groups:** In the `pattern` part, enclose the parts you want to capture within escaped parentheses: `\(...\)`.
- **Referencing Capture Groups:** In the `replacement` part, refer to the captured groups using backreferences: `\1`, `\2`, etc., corresponding to the order of the capture groups in the pattern. `\0` or `&` refers to the entire matched text.

```sh
:%s/\(foo\) \(bar\)/\2 \1/g
```

```
:%s/original/replacement/g(for global)c(ask)
```
where `.` could mean any character
The `%` makes it search the entire buffer instead of just the current line


# Quickfix
The results of `:vimgrep` are populated into Vim's quickfix list. This list allows for easy navigation between the search hits.

- `:copen`: Opens the quickfix window.
- `:cnext` / `:cn`: Jumps to the next entry in the quickfix list.
- `:cprevious` / `:cp`: Jumps to the previous entry in the quickfix list.
- `:cclose`: Closes the quickfix window.