#cheatsheet 
# movement
- `hjkl` ⇒ move
	- `J`: put the next line on the current
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
- `%` ⇒ jump to the matching character `(){}[]`
- `gg` ⇒ jump to the beginning of a buffer
    - G ⇒ jump to the end
- `gr` ⇒ go to reference
- `gd` ⇒ go to local declaration
- `gD` ⇒ go to global declaration
- `gI` ⇒ go to implementation
- `g]` ⇒ go to definition
- `gr` ⇒ go to reference
- `C-o` ⇒ return to the previous position _**(IN A JUMP LIST (history))**_
- `C-e` ⇒ move window down
    - `C + y `⇒ up
    - `C-d `⇒ 1/2 window down
    - `C-u `⇒ 1/2 window up
    - `C-f `⇒ one window down
    - `C-b `⇒ one window up
    - `C-h` ⇒ move to _left window_
        - `C-l `⇒ move to _right window_
- `H` ⇒ jump to the first line in window
    - `L` ⇒ end
    - `M` ⇒ middle
- `zz` ⇒ center a window around cursor
- - ⇒ jump next appearance under the cursor
        - `#` ⇒ jump to previous
- `n` ⇒ repeat search
    - N ⇒ repeat search opposite direction
- `f[+ any character]`⇒ jump to the character in the line
	- `F[+ any character]` => reverse direction
- `C-w <hjkl>`: move between windows


# visual
- `v`: insert visual mode
- `ctrl+v`: **visual block**, so now you can add many cursors (apply your work on multiple lines)
	- `I`: allows to insert for **all** lines
- **u** - change marked text to lowercase
- **U** - change marked text to uppercase


# insert
- `i` ⇒ insert before cursor mode
    - I ⇒ insert mode to the beginning of a line
- `a` ⇒ append after cursor
    - A ⇒ append to the end of a line
    - ea ⇒ append the end of a word
- `o` ⇒ append a new line below
    - `O` ⇒ append above
# editing (after that we go to insert)
-  `r` ⇒ replace one character
        - R ⇒ replace until ESC
- `J` ⇒ join 2 lines with a space between
- `u` ⇒ undo
- `C-r` ⇒ redo
- . ⇒ repeat last line command
- `gu` ⇒ turn to lowercase
- `gU` ⇒ turn to uppercase
- `cc` ⇒ change (replace) entire line
	- `c$ `or `C` ⇒ change from cursor to the end of the line
- `ciw` ⇒ change, insert, word ⇒ replace entire word
- `cib` ⇒ change, insert, brackets ⇒ replace the contents of a brackets
	- `ci+shift+b`: c inside curly braces
- `ci`{ ⇒ change the contents inside {}
- `s` ⇒ delete character AND substitute (replace) character
	- S ⇒ equivalent to cc
- `xp` ⇒ transpose 2 letters (delete and paste)


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


# formatting
- `=`: format indentation


```go
func main() {
	fmt.Println()
}
```