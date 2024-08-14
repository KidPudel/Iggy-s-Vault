#cheatsheet
# Obsidian
- [[]]: creates a new link
- `cmd + N`: create a note
- `cmd + shift + N`: create a note on a right side
- `cmd+T`: new tab
- `cmd + K`: insert a markdown link
- `cmd + G`: open a graph
- `cmd + D`: delete graph
- `cmd + P`: open command palette
- `cmd + W`: close tab
- `opt+enter`: follow link under cursor
- `cmd + enter`: open link under cursor in a new tab
	- `cmd+opt+shift+enter`: open link in a new window
-  `opt + tab`/`cmd+shift+]`: go to the next tab
- `opt+shift+ ->`: go forward
	- go backwards
- `cmd+,`: open setting
- `cmd+O`: [quick switcher](https://help.obsidian.md/Plugins/Quick+switcher)
- `cmd+opt+F`: search & replace in current file
- `cmd+shift+F`: global search
- `cmd+L:` toggle checkbox

# vscode/zed

- `cmd+shift+p`: command palette/show all commands
- `cmd+p`: go to file
- `cmd+t`: go to symbol in workspace
- `cmd+shift+o`: go to symbol in editor/buffer
- `ctrl+space`: completion suggestion
- `cmd+k+cmd+i`: quick documentation lookup
- `shift+cmd+space`: show parameters
- `cmd+j`: toggle panel
- `ctrl+tilda` : toggle terminal (secondary side bar)
- `ctrl+~`: new terminal
- `cmd+b`: toggle left side bar
- `cmd+r`: togger right side bar
- `ctrl+shift+tilda`: create new terminal
- `cmd+\`: split window
- `cmd+w`: close window
- `cmd+k + arrows (<- ->)`: move between split views
- `opt+cmd+[]`: Fold and unfold
- `cmd+shift+o`: search symbol in buffer
- `cmd+k+z`: zen mode
- `cmd+k+l`: minimap
- `cmd+shift+x`: extensions
- `cmd+shift+e`: explorer
- `opt+shift+f`: reformat
- `option+enter`: generate...
- `cmd+.`: quick fix
- `cmd+k + r`: reveal in finder
- `cmd+opt+b`: git switch branches
- `cmd+opt+g +b`: git blame
- `cmd+g`: next match
- `cmd+shift+g`: previous match
- `cmd+opt+f`: replace
- `cmd+shift+f`: global search
- `cmd+shift+m`: errors multibuffer
- `cmd+shift+i`: format


# Vim
## movement
- `hjkl` ⇒ move
- `w` ⇒ jump the word
    - `e` ⇒ jump the en
    - d of a word
        - ge ⇒ jump back to the end of a word
    - `b` ⇒ jump back one word
- `$` ⇒ jump the end of a line
    - ^ ⇒ jump beginning of a characters line
    - 0 ⇒ jump beginning of a line
- `}` ⇒ jump to the next paragraph
    - `{` ⇒ previous
- `[{ `or `[< `or `[(` ⇒ jump to the beginning of a block surrounded by braces
    - `]` ⇒ to the end
- `] + m` (start) M (end) ⇒ go to the next function
    - `[ + m` ⇒ previous
- `%` ⇒ jump to the matching character (){}[]
- `gg` ⇒ jump to the beginning of a buffer
    - G ⇒ jump to the end
- `gr` ⇒ go to reference
- `gd` ⇒ go to local declaration
- `gD` ⇒ go to global declaration
- `gI` ⇒ go to implementation
- `g]` ⇒ go to definition
- `gr` ⇒ go to reference
- `control + o` ⇒ return to the previous position _**(IN A JUMP LIST (history))**_
- `control + e` ⇒ move window down
    - `control + y `⇒ up
    - `control + d `⇒ 1/2 window down
    - `control + u `⇒ 1/2 window up
    - `control + f `⇒ one window down
    - `control + b `⇒ one window up
    - `control + h` ⇒ move to _left window_
        - `control + l `⇒ move to _right window_
- `H` ⇒ jump to the first line in window
    - `L` ⇒ end
    - `M` ⇒ middle
- `zz` ⇒ center a window around cursor
- - ⇒ jump next appearance under the cursor
        - `#` ⇒ jump to previous
- `n` ⇒ repeat search
    - N ⇒ repeat search opposite direction
- `f[+ any character]`⇒ jump to the character in the line


- `ctrl+v`: visual block, so now you can add many cursors 


## insert
- `i` ⇒ insert before cursor mode
    - I ⇒ insert mode to the beginning of a line
- `a` ⇒ append after cursor
    - A ⇒ append to the end of a line
    - ea ⇒ append the end of a word
- `o` ⇒ append a new line below
    - `O` ⇒ append above
## editing (after that we go to insert)
-  r ⇒ replace one character
        - R ⇒ replace until ESC
    - J ⇒ join 2 lines with a space between
    - u ⇒ undo
    - control + r ⇒ redo
    - . ⇒ repeat last line command
    - gu ⇒ turn to lowercase
    - gU ⇒ turn to uppercase
    - cc ⇒ change (replace) entire line
        - c$ or C ⇒ change from cursor to the end of the line
    - ciw ⇒ change, insert, word ⇒ replace entire word
    - cib ⇒ change, insert, brackets ⇒ replace the contents of a brackets
    - ci{ ⇒ change the contents inside {}
    - s ⇒ delete character AND substitute (replace) character
        - S ⇒ equivalent to cc
    - xp ⇒ transpose 2 letters (delete and paste)
    - **u** - change marked text to lowercase
    - **U** - change marked text to uppercase

## cut and paste (we don’t go to insert)

- “+y ⇒ yank to the **SYSTEM clipboard**
-  yy ⇒ yank (copy) a line
- nyy (where n is number) ⇒ yank n lines
- yw ⇒ yank word
- y$ or Y ⇒ yank to the end of a line
- p ⇒ paste the clipboard after cursor
- “+p ⇒ paster from **SYSTEM clipboard**    
- P ⇒ before
- dd ⇒ delete (cut) a line
- ndd ⇒ n lines
- diw ⇒ delete (cut) a word        
- dw ⇒ delete from cursor to the end of the word
- :3, 5d ⇒ delete (cut) lines 3-5 
- x ⇒ delete (cut) a character
# Apple notes
- `shift+cmd+7`: bullet point

# Apple reminders
- `cmd+e`: show nested
- `cmd+opt+s`: show/hide sidebar
# Warp
- `cmd+T`: new tab
- `cmd+,`: settings
- `ctrl+shift+R`: workflows
- `cmd+\`: warp drive


# MacOs
- `cmd+opt+d`: hide the dock
- `ctrl+cmd+f`: full screen
- `ctrl + -> <-`: switch between full-screen windows
- `ctrl+cmd+d`: view word definition
- `shift+cmd+g`: go to folder (finder)

# Arc
- `cmd+shift+d`: move url to the main view
- `opt+click`: opens new on side
- `ctrl+shift++`: new side bar
