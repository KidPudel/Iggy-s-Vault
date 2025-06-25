unsafe low-level [[bitwise]] casting where you are telling to compile to trust you
sizes must match exactly
- Use `cast` when you want type-safe conversions.
- Use `transmute` only when you **know** that two types are bit-compatible and need to reinterpret data (e.g., for performance hacks, unions, packed structs).