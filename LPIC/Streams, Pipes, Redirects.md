STDIN 
STDOUT
STDERR
- `>` Redirect STDOUT to a file; Overwrite if exists
- `>>` Redirect STDOUT to a file; Append if exists
- `2>` Redirect STDERR to a file; Overwrite if exists
- `2>>` Redirect STDERR to a file; Append if exists
- `&>` Redirect both STDOUT and STDERR; Overwrite if exists
- `&>>` Redirect both STDOUT and STDERR; Append if exists
- `<` Redirect STDIN from a file
- `<>` Redirect STDIN from the file and send the STDOUT to it
- It is also possible to use `&1` and `&2` and `&0` to refer to the **target** of STDOUT, STDERR & STDIN. In this case `ls > file1 2>&1` means redirect output to file1 and output stderr to same place as stdout (file1)
Sending to null
- **/dev/null** device works like an abyss