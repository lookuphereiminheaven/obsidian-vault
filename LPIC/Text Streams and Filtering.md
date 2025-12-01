Viewing commands
`cat` 
- `-n` show line numbers
- `-s` squeeze blanks
- `-T` show tabs
- `-v` shows nonprinting char
---
`less` to view larger text files
- `q` exit
- `/foo` searches for foo
- `n` Next (search)
- `N` Previous (search)
- `?foo` search backward for foo
- `G` go to end
- `nG` Go to line n
---
`od` Shows files in formats other than text
- `-t a` or `-a` showing only named characters
- `-t c` or `-c` showing escaped chars
- `-A d` Decimal
- `-A o` Octal
- `-A x` Hex
- `-A n` None
---
`split` transferring HUGE files on smaller media
- `split -l 2 mydata output` split mydata into outputaa, outputab, ...; 2 lines per file
- the `-l 2` splits 2 lines per file. It is possible to use `-b 42` to split every 42 bytes or even `-n 5` to force 5 output files
- --
`head` and `tail` shiws beggining and end of text files
- default is 10 lines but change it using `-n20`
---
`cut` cut one or more columns from a file
- Default delimiter is TAB. use `-dx` to change it to "x" or `-d' '` to change it to space
---
 `nl` shows line numbers
 `sort`
 `uniq` deduplicate
 `paste` pastes lines from two or more files side-by-side
 `tr` _translates_ characters in the stream
 `sed 's/A/B/'` replace once in each line
 `wc` wordcount count line n umbers with `-l`

 
