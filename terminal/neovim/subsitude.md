# Vim Substitution Deep Dive

The substitute command is one of Vim's most powerful features. Here's what actually matters:

## Basic syntax

```vim
:[range]s/pattern/replacement/[flags]
```

**Range:**

- `%` = entire file (most common)
- `.` = current line
- `'<,'>` = visual selection (auto-inserted when you select and type `:`)
- `1,10` = lines 1 to 10
- `.,$` = current line to end of file
- `.,+5` = current line plus next 5 lines

**Flags:**

- `g` = all occurrences on line (without this, only first match per line)
- `c` = confirm each substitution
- `i` = case insensitive
- `I` = case sensitive (override ignorecase setting)
- `n` = report number of matches, don't actually substitute

---
## Patterns you'll actually use

### Literal text (no regex)

```vim
:%s/old/new/g
:%s/foo bar/baz qux/g
```

---
### Escape special characters

These need escaping: `. * [ ] ^ $ / \`

```vim
:%s/example\.com/newsite.com/g
:%s/\*/replacement/g
:%s/\[old\]/[new]/g
```

**Pro tip:** Use a different delimiter to avoid escaping slashes:

```vim
:%s#/path/to/old#/path/to/new#g
:%s@old/path@new/path@g
```

---
### Word boundaries

```vim
:%s/\<word\>/replacement/g    " matches 'word' but not 'password'
:%s/\<id\>/userId/g            " matches 'id' not 'hidden'
```

---

### Case variations

```vim
:%s/\cfoo/bar/g               " case insensitive (foo, Foo, FOO all match)
:%s/foo\|Foo\|FOO/bar/g       " explicit alternation
```

---

### Capture groups (THE IMPORTANT ONE)

Use `\(` and `\)` to capture, `\1`, `\2`, etc. to reference:

```vim
" Swap two words
:%s/\(\w\+\) \(\w\+\)/\2 \1/g
" "hello world" → "world hello"

" Wrap function names in backticks
:%s/\(\w\+\)(/`\1`(/g
" "myFunc()" → "`myFunc`()"

" Extract domain from email
:%s/.*@\(.*\)/\1/g
" "user@example.com" → "example.com"

" Add quotes around words
:%s/\(\w\+\)/"\1"/g
" "hello world" → ""hello" "world""

" Swap CSS property-value
:%s/\(.*\): \(.*\);/\2: \1;/g
" "color: red;" → "red: color;"
```

---
### Character classes

```vim
\d    " digit [0-9]
\D    " non-digit
\w    " word character [a-zA-Z0-9_]
\W    " non-word character
\s    " whitespace
\S    " non-whitespace
.     " any character
```

**Examples:**

```vim
:%s/\d\+/NUMBER/g              " replace all numbers
:%s/\s\+$//g                   " trim trailing whitespace
:%s/\w\+/\U&/g                 " uppercase all words (see below)
```

---
### Quantifiers

```vim
*     " 0 or more (greedy)
\+    " 1 or more (greedy)
\?    " 0 or 1
\{n}  " exactly n times
\{n,} " n or more times
\{n,m}" between n and m times
```

**Examples:**

```vim
:%s/a\+/A/g                    " "aaa" → "A", "aa" → "A"
:%s/\d\{3\}/XXX/g              " replace 3-digit numbers
:%s/x\{2,4\}/Y/g               " "xx", "xxx", "xxxx" → "Y"
```

---
### Beginning and end

```vim
^     " start of line
$     " end of line
```

**Examples:**

```vim
:%s/^/# /g                     " comment every line with #
:%s/^    //g                   " remove 4-space indentation
:%s/;$//g                      " remove trailing semicolons
:%s/^$\n//g                    " delete empty lines
```

---
## Replacement tricks

### Special replacement characters

```vim
&     " the whole matched pattern
\0    " same as & (the whole match)
\1-\9 " captured groups
\u    " uppercase next character
\U    " uppercase until \E
\l    " lowercase next character  
\L    " lowercase until \E
\E    " end \U or \L
```

**Examples:**

```vim
:%s/\w\+/\u&/g                 " Capitalize first letter of each word
:%s/\w\+/\U&/g                 " UPPERCASE ALL WORDS
:%s/\(.\)/\u\1/g               " Capitalize first letter of line

" Convert snake_case to camelCase
:%s/_\(\w\)/\u\1/g
" "my_variable_name" → "myVariableName"

" Convert camelCase to snake_case
:%s/\([a-z]\)\([A-Z]\)/\1_\l\2/g
" "myVariableName" → "my_variable_name"
```

### Use expressions in replacement

Use `\=` to evaluate Vimscript:

```vim
:%s/\d\+/\=submatch(0)*2/g     " double all numbers
:%s/$/\=line('.')/g            " add line number at end of each line
:%s/\w\+/\=strlen(submatch(0))/g  " replace words with their length
```

---
## Patterns I use constantly

**Remove trailing whitespace:**

```vim
:%s/\s\+$//g
```

**Delete blank lines:**

```vim
:g/^$/d
```

(This uses `:g` global command, not substitute, but related)

**Add semicolon to end of lines missing one:**

```vim
:%s/[^;]$/&;/g
```

**Convert spaces to tabs:**

```vim
:%s/    /\t/g
```

**Remove all numbers:**

```vim
:%s/\d\+//g
```

**Swap function arguments:**

```vim
:%s/func(\(.*\), \(.*\))/func(\2, \1)/g
```

**Extract URLs from markdown links:**

```vim
:%s/\[.*\](\(.*\))/\1/g
" "[text](url)" → "url"
```

**Convert list to comma-separated:**

```vim
:%s/\n/, /g
```

## Very magic mode

Add `\v` to use "very magic" mode (more like normal regex):

```vim
:%s/\v(foo|bar)/baz/g          " alternation without escaping
:%s/\v\w+@\w+\.com/EMAIL/g     " cleaner email pattern
:%s/\v^(\s+)/\1\1/g            " double indentation
```

**Without `\v` you'd write:**

```vim
:%s/\(foo\|bar\)/baz/g
```

I use `\v` for complex patterns, skip it for simple ones.

## Interactive workflow

**Most common pattern:**

```vim
:%s/old/new/gc
```

Then for each match:

- `y` = yes, replace
- `n` = no, skip
- `a` = all, replace remaining without asking
- `q` = quit
- `l` = replace this one and quit

**Search first, then substitute:**

```vim
/pattern        " see all matches highlighted
:%s//new/g      " empty pattern reuses last search
```

This is huge for verifying your pattern works before replacing.

## Common gotchas

**Greedy matching:**

```vim
:%s/<.*>//g
```

On `<div>text</div>` this removes EVERYTHING, not just tags.

**Fix with non-greedy:**

```vim
:%s/<.\{-}>//g
```

Now it correctly removes `<div>` and `</div>` separately.

**Case sensitivity:** Vim respects your `ignorecase` setting. Override with `\c` (ignore) or `\C` (match):

```vim
:%s/\cword/replacement/g      " case insensitive
:%s/\Cword/replacement/g      " case sensitive
```

## Global command (bonus)

Not substitute, but related and super useful:

```vim
:g/pattern/d                   " delete lines matching pattern
:v/pattern/d                   " delete lines NOT matching (inverse)
:g/TODO/norm A !!              " append !! to lines with TODO
:g/^import/m0                  " move all import lines to top
```

**Examples:**

```vim
:g/console.log/d               " delete all console.log lines
:v/^#/d                        " keep only lines starting with #
:g/FIXME/t$                    " copy all FIXME lines to end of file
```

## Practice exercises

Try these on real files:

1. **Remove comments:** `:%s/\s*\/\/.*$//g`
2. **Add quotes:** `:%s/\w\+/"\0"/g`
3. **Swap names:** `:%s/\(\w\+\), \(\w\+\)/\2 \1/g` (Last, First → First Last)
4. **Increment numbers:** `:%s/\d\+/\=submatch(0)+1/g`
5. **Extract domain:** `:%s/https\?:\/\/\([^/]\+\).*/\1/g`

## What to actually memorize

**Core:**

- `:%s/old/new/g` and `:%s/old/new/gc`
- Word boundaries: `\<` and `\>`
- Capture groups: `\(pattern\)` and `\1`
- Character classes: `\d`, `\w`, `\s`
- Quantifiers: `*`, `\+`, `\?`

**Level 2:**

- Case conversion: `\u`, `\U`, `\l`, `\L`
- Very magic mode: `\v`
- Non-greedy: `.\{-}`
- Special replacements: `&`, `\0`

**Everything else:** Look up when needed. Don't try to memorize all of regex.

## Resources

**Test patterns:**

```vim
:echo matchstr("test string", 'pattern')    " test pattern matching
:echo substitute("test", 'pattern', 'replacement', 'g')  " test substitution
```

**Help:**

```vim
:h :s
:h pattern
:h sub-replace-special
```

The real skill isn't memorizing patterns—it's recognizing "this is a job for substitute" and then looking up the exact pattern syntax. Keep a cheat sheet handy for the first few weeks.

Want me to walk through any specific pattern or use case?