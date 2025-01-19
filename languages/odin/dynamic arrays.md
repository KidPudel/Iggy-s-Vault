Dynamic arrays are similar to [[slices]] in odin, except that its length is may change during runtime.
They are similar to slices in go. read [[slices under the hood]]

```odin
Raw_Dynamic_Array :: struct {
	data:      rawptr,
	len:       int,
	cap:       int,
	allocator: Allocator,
}
```

So it is a struct and nothing more


```odin
package main

import "core:fmt"
import "core:mem"

main :: proc() {
	s := make([dynamic]int, 0, 4)
	// point to backing array, just like in Golang, but instead of keeping up to date by returning to a new holder, we change with reference to the same
	append(&s, ..[]int{1, 2, 3})

	// to replicate go's slice behavior, we need to remember
	// append changes underlying array, and returns an updated new struct

	// create a new struct, with the same content
	s1 := s
	append(&s1, 4)

	s2 := s
	append(&s2, 5) // override

	s3 := s
	append(&s3, 6, 7) // allocates a new underlying array and kills the previous

	fmt.println(s, s1, s2, s3)
}

```



```lua
	{
		"rebelot/kanagawa.nvim",
		config = function()
			require("kanagawa").setup({
				keywordStyle = { italic = false },
				overrides = function(colors)
					local palette = colors.palette
					return {
						String = { italic = true },
						Boolean = { fg = palette.dragonPink },
						Constant = { fg = palette.dragonPink },

						Identifier = { fg = palette.dragonBlue },
						Statement = { fg = palette.dragonBlue },
						Operator = { fg = palette.dragonGray2 },
						Keyword = { fg = palette.dragonRed },
						Function = { fg = palette.dragonGreen },

						Type = { fg = palette.dragonYellow },

						Special = { fg = palette.dragonOrange },

						["@lsp.typemod.function.readonly"] = { fg = palette.dragonBlue },
						["@variable.member"] = { fg = palette.dragonBlue },
					}
				end,
			})
			vim.cmd("colorscheme kanagawa-dragon")
		end,
	},

```