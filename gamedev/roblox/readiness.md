Because Roblox can’t promise that everything is available at once, **your logic must be readiness-based and incremental**, which is why the correct patterns are:
- WaitForChild
- GetDescendants() + DescendantAdded
- “Register handlers when objects show up”
- Assume objects may appear or disappear at any time

[[Roblox APIs]]