## Vim Modes
Vim has 4 modes
- `normal mode` - Its for navigation and commands
- `insert mode` - Its for typing text
- `visual mode` - Its for selecting text before performing an operation. 
- `command mode` - Its for executing commands. Its triggered via semicolon 
## Basic VIM motions
- h,j,k,l (left, down, up, right)
- `o / O` - Create a new line after/before cursor and go to insert mode. 
- `a` - Go to insert mode and insert after the cursor.
- `A` - Go to end of line and go to insert mode.
- `w/b` - go forward word/backward by word
## Basic VIM commands
- `d` - delete`
- `p` - paste
- `y` - yank (copy)
- `u` - Undo
- `Ctr + R` - Redo
- `x` - cut (by default, cut/copy/paste throw characters in the same buffer) 

### Anatomy of a vim motion
vim keysets can be composed by the following -
`<command> <number> <motion>
- eg to delete 5 lines from current position to top, we can do `d5j`

## Vim horizontal Motions
- 