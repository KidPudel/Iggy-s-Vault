extremely useful to find and manipulate files

```sh
find [path] [options] [tests] [actions]
```

```sh
find ~/dotfiles -name '.DS_Store' -type f -delete
```

# Common tests
```sh
-name "pattern"          # Match filename (case-sensitive)
-iname "pattern"         # Match filename (case-insensitive)
-type f                  # Files only
-type d                  # Directories only
-size +10M               # Larger than 10MB
-size -1k                # Smaller than 1KB
-mtime -7                # Modified within last 7 days
-mtime +30               # Modified more than 30 days ago
-user username           # Owned by user
-perm 755                # Exact permissions
-empty                   # Empty files/directories
```

# Common actions
```sh
-delete                  # Delete matches (USE CAREFULLY)
-exec cmd {} \;          # Execute command on each match
-exec cmd {} +           # Execute command on all matches at once
-print                   # Print path (default)
-ls                      # Detailed listing
```


# Practical commands
```sh
# Find all .txt files
find . -name "*.txt"

# Find files modified in last 24 hours
find . -type f -mtime -1

# Find and delete empty files
find . -type f -empty -delete

# Find large files (>100MB)
find / -type f -size +100M

# Find and execute command
find . -name "*.log" -exec rm {} \;

# Case-insensitive search
find . -iname "readme*"

# Multiple conditions (AND)
find . -type f -name "*.py" -size +1M

# OR condition
find . -name "*.jpg" -o -name "*.png"

# Exclude directory
find . -path ./node_modules -prune -o -name "*.js" -print

# Find out how many lines in a file
find . -name "pack.lua" -exec wc {} +
```

# Operators
```sh
-not or !                # Negation
-and                     # AND (default)
-or or -o                # OR
\( \)                    # Grouping
```