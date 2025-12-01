 `*` **any string**
 `?` any single character
 `[ABC]` matches A, B, or C
 `[a-k]` matches a, b, c, ..., k (both lower-case and upper-case)
 `[0-9a-z]` matches all digits and numbers
`[!x]` means NOT X.
`ls` lists directories
- `-l` longer info
- `-1` prints 1 file per line
- `-t` sorts based on modification date
- `-r` reverses search
  `-tr` reverse times (newer files at the bottom)
- `-ltrh` long + human readable size + reverse time
`cp` copy
- `-r` or `-R` copies directories and contents
`rm` remove
- `-f` force
  `-i` interactive yes or no questions
- `-b` backup
- `-p` preserve attributes
`mv` move
`mkdir` create directory
- `-p` create parent directory and trees of directories
`rmdir` remove directory
- `-p` nested removing
`touch`will create an empty file (if it does not exists) or updates the **modification** date of a file if it already exist
- `-d` give date
- `-t` give timestamp
- `-r` reference to use another file's time
`file` determines file type
- `-i` switch prints the mime format
`dd` copies data from its input to its output or read/write from block devices
- `if` input file
- `of` output file
`find` find files
- `find . -iname "[a-j]*"` 
- The first parameter says where should we search (including subdirectories).
- The `-name` switch indicates the criteria (here `iname` means search files with this name and ignore the character cases (z equals Z)).
- `-size 100c` files which are exactly 100 characters/bytes (you can also use `b`)
- `-size +100k` files which are more than 100 kilobytes
- `-size -20M` files smaller than 20Megabytes
- `-size +2G` files bigger than 2Gigabytes
- `find /var -iname '*tmp' -size +1M -size -100M
- find all empty files with `find . -size 0b` or `find . -empty`
Time switches
![[Pasted image 20251201133431.png]]
- if we add `-daystart` switch to -`mtime` or `-atime` it means that we want to consider days as calendar days, starting at midnight
Acting on files
- `ls` will run `ls -dils` on each file
- `-print` will print full name of files on each line
- use `-exec` switch and `'{}'` or `{}` to point to file and finish with `\;`
- `find . -name "*.htm" -exec mv '{}' '{}l' \;` renames all .htm files to .html
- `-delete` also deletes
Compression
`gzip` and `gunzip`
- preserves time
- creates new compressed file with same name but .gz ending
- removes the original files after creating the compressed file (you can keep the input file with `-k` switch)
`bzip2` and `bunzip2`
- different compression algorithm
`xz` and `unxz`
- another compression tool
- _ompressing_ a small text file makes it larger. This is _normal_ in small files because of all the headers and metadata
- commands like `unxz` is just a calls to `xz --dcompress`
Archiving
`tar` automatically creates an archive file from a directory and all its subdirs
- `-cf` create tar file
- `-xf` extract a tar
- `-z` compress with `gzip` after creation
- `-j` compress with `bzip2` 
- `-v` verbose
- `-r` append new files to the archive
`cpio`
- gets lists of files and creates one archive
- `-o` create an output
- `-d` create folders
- `-i` for extract
```
mkdir extract mv myarchivefind.cpio extract cd extract cpio -id < myarchivefind.cpio
```
