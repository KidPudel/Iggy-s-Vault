vim-fugitive

1. `:Gvdiffsplit!`, to open 3 pane view, be on the you target branch (middle)
2. and choose which side you want to pick left (2) or right (3) with `:diffget`
3.  or use `:diffput` which is the opposite, we need to stand on one of the branches, and put into the target
4. then you need to stage changes, a hunk or whole buffer with `:Gitsigns`
5. and then continue rebasing you changes with `:Git rebase --continue`!
you are done!