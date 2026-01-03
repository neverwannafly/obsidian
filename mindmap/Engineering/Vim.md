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
- `I` - insert mode at the beginning of the line
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
- `f + <char>` will lead to the space exactly at that character in left to right fashion. Capital F will do that in reverse
- `t + <char>` will lead to the space one place behind that character in left to right fashion. Capital T will do that in reverse
- `;` and `,` will cycle through the results on the same line 
