# VIM

# General management

* `vimtutor` in a terminal to learn the first Vim commands
* `:h[elp] keyword` - open help for keyword
* `sav[eas] file` - save file as
* `:clo[se]` - close current pane
* `:ter[minal]` - open a terminal window
* `K` - open man page for word under the cursor

# Moving (insert mode)

## Simple

* `e` `h`, `j`, `k` - arrows 

## Word

* `w`, `e`, `b` - word movements (MAIUSC for space-separated, punctuation part of word)

## Line

* `0`, `^`, `$` - start of line, first non-blank character, end of line
* `f`, `t` - find in line (MAIUSC for backwards, `;` and `,` to move)

## Block

* `(`, `)` - previous, next sentence
* `{`, `}` - previous, next paragraph (or function block) 
* `%` - matching bracket

## Screen

* `H`, `M`, `L` - high, middle, low position of screen

## Document

* `gg`, `G` - top, bottom of document
* `/`, `?` - find in file (then `n` and `N` to move) 

# Enter insert mode

* `i`, `a` - insert, append (MAIUSC for "line-wise")
* `o`, `O` - line above, line below

# Exit insert mode (return to normal mode)

* `Esc`, `CTRL+C`, `CTRL+[` - return to normal mode 
  * `:imap jj <ESC>` - map two subsequent presses of j to be interpreted as ESC

# Indent

* `Ctrl+t`, `Ctrl + d` - indent, de-indent line in insert mode
* `>>`, `>>` - indent, de-indent line in normal mode


