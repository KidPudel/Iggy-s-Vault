In go string is actually an array of **bytes**, so if it uses utf8 then it could give the amount of bytes if using `len()`, so use `utf8` package
we can convert to slice or runes which are int32 alias