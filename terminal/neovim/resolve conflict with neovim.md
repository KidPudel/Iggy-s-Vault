vim-fugitive

1. `:Gvdiffsplit!`, to open 3 pane view, be on the you target branch (middle)
2. and choose which side you want to pick left (2) or right (3) with `:diffget`
3. then you need to stage changes, a hunk or whole buffer with `:Gitsigns`
4. and then continue rebasing you changes with `:Git rebase --continue`!
you are done!