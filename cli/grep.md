#cli
filter a data in searching

find a match in a file
```
grep "string we want to find" file.txt
```

recursively search for a match
```
grep -r "match" *
```


## flags
- `-r`: recursively
- `-i`: case insensitive
- `-c`: count matches
- `-v` invert output (that does not match)
- `-L`: files that does not contain a match
- `-n`: add number of lines for the result
- `-w`: exact search
- `-l`: lowercase
- `-I`: uppercase

## Usage with a pipe operator
 For example, If you want to know if a certain package is installed in Ubuntu system execute

```bash
$ dpkg -L | grep "package-name"
```

[more info](https://www.digitalocean.com/community/tutorials/grep-command-in-linux-unix)
