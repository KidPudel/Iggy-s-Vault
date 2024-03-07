#cli
sed is stream editor, used to perform basic text transformation

Normally `sed` is invoked like this:

```bash
sed SCRIPT INPUTFILE...
```

For example, to replace all occurrences of ‘hello’ to ‘world’ in the file input.txt:

```bash
sed 's/hello/world/' input.txt > output.txt
```
> Note: by default sed writes output to the standard output

```bash
echo "test" | sed -n "w test.md"
```

## tags
- `-i`: edit file *in-place*, instead of printing to the standard output
```bash
sed 's/hello/bye' file.txt
```
- `-n`: suppress output


## commands
- `p`: write specific line of the file
```bash
grep '45p' file.txt
```
> Writes 45th line of the input file
- `s`: for substitute, is the most important command in sed (syntax of the `s` command is ‘s/regexp/replacement/flags’.)
- `a\ text`: append text after a line
- `w`: write the pattern space to filename


[more info on commands]()



---

[more info](https://www.gnu.org/software/sed/manual/sed.html)