# SOLID in Go

SOLID applied to Go using interfaces and composition.

Go does not have classes or classical inheritance, but SOLID principles apply via interfaces, struct composition, and package design.

## Principles

- [[single-responsibility-principle]]
- [[open-closed-principle]]
- [[liskov-substitution-principle]]
- [[interface-segregation-principle]]
- [[dependency-inversion-principle]]

## Go-specific notes

Go interfaces are implemented implicitly — a type satisfies an interface without declaring it. This naturally supports OCP and ISP: define small interfaces, implement them anywhere.

DIP is achieved by accepting interfaces as parameters rather than concrete types. Go has no keyword for dependency injection; it's done via constructor functions.
