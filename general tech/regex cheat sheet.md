# Regex Reference

## Basic Patterns

```regex
.           # Any single character (except newline)
\d          # Digit [0-9]
\D          # Non-digit
\w          # Word character [a-zA-Z0-9_]
\W          # Non-word character
\s          # Whitespace [ \t\n\r\f]
\S          # Non-whitespace
^           # Start of string/line
$           # End of string/line
\b          # Word boundary
\B          # Non-word boundary
```

## Quantifiers

```regex
*           # 0 or more (greedy)
+           # 1 or more (greedy)
?           # 0 or 1 (optional)
{3}         # Exactly 3
{3,}        # 3 or more
{3,7}       # Between 3 and 7
*?          # 0 or more (lazy/non-greedy)
+?          # 1 or more (lazy)
??          # 0 or 1 (lazy)
```

## Character Classes

```regex
[abc]       # Any of a, b, or c
[^abc]      # NOT a, b, or c
[a-z]       # Range a through z
[a-zA-Z]    # Any letter
[0-9]       # Any digit
[^0-9]      # Not a digit
```

## Groups & Alternation

```regex
(abc)       # Capture group
(?:abc)     # Non-capturing group
(?<name>abc) # Named capture group
|           # OR operator
(a|b)       # Match a or b
```

## Anchors & Boundaries

```regex
^start      # Must start with "start"
end$        # Must end with "end"
^exact$     # Exact match only
\bword\b    # Whole word "word"
```

## Escaping Special Characters

```regex
\.  \*  \+  \?  \[  \]  \(  \)  \{  \}  \^  \$  \|  \\
```

## Lookahead & Lookbehind

```regex
(?=...)     # Positive lookahead
(?!...)     # Negative lookahead
(?<=...)    # Positive lookbehind
(?<!...)    # Negative lookbehind
```

## Practical Examples

**Email:**

```regex
\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b
```

**Phone (US):**

```regex
\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}
```

**URL:**

```regex
https?://[^\s]+
```

**IP Address:**

```regex
\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b
```

**Date (YYYY-MM-DD):**

```regex
\d{4}-\d{2}-\d{2}
```

**Hex Color:**

```regex
#[0-9A-Fa-f]{6}
```

**Extract text between quotes:**

```regex
"([^"]*)"
```

**Match word not followed by number:**

```regex
\b\w+\b(?!\d)
```

## Common Patterns

**Username (alphanumeric + underscore, 3-16 chars):**

```regex
^[a-zA-Z0-9_]{3,16}$
```

**Strong password (8+ chars, upper, lower, digit, special):**

```regex
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$
```

**Remove extra whitespace:**

```regex
\s+          # Replace with single space
```

**Match lines NOT containing "error":**

```regex
^(?!.*error).*$
```

## Flags/Modifiers

```
i     # Case-insensitive
g     # Global (all matches)
m     # Multiline (^ and $ match line starts/ends)
s     # Dot matches newline
x     # Ignore whitespace in pattern
```

## Greedy vs Lazy

**Greedy (default):**

```regex
<.+>        # Matches: <a>text</a>  →  "<a>text</a>"
```

**Lazy:**

```regex
<.+?>       # Matches: <a>text</a>  →  "<a>" and "</a>"
```

## Testing Tools

- regex101.com (best for learning, shows explanations)
- regexr.com
- Your CLI: `grep -E`, `sed`, `awk`

## Quick Tips

1. **Start simple, add complexity** - Build patterns incrementally
2. **Test edge cases** - Empty strings, special chars, boundaries
3. **Use raw strings** in code - `r"\d+"` (Python) to avoid escape hell
4. **Capture only what you need** - Use `(?:...)` for non-capturing groups
5. **Anchors matter** - `^` and `$` prevent partial matches