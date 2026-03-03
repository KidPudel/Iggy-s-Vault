# Go Naming Conventions

The standard naming rules for identifiers in Go code.

## What it does

Go uses capitalization to control export visibility. An identifier starting with an uppercase letter is exported (public); lowercase is unexported (package-private). There are no keywords for access modifiers.

| Identifier     | Convention                             | Example                |
| -------------- | -------------------------------------- | ---------------------- |
| Exported       | PascalCase                             | `UserService`          |
| Unexported     | camelCase                              | `userService`          |
| Constants      | SCREAMING_SNAKE_CASE or MixedCaps      | `INT_MAX`, `MaxSize`   |
| Bool vars      | Prefix with `Is`, `Has`, `Can`, `Allow`| `isReady`, `hasError`  |
| Interfaces     | Noun + `-er` suffix                    | `Reader`, `Writer`     |
| Packages       | Single lowercase word, no underscores  | `json`, `http`         |
| Files          | lowercase, underscore-separated        | `user_service.go`      |
| Test files     | `_test.go` suffix                      | `user_service_test.go` |

## Sources

- https://go.dev/doc/effective_go#names

