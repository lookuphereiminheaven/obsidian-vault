check where your general `sh` command links to
- `readlink /bin/sh`
check your `$SHELL` variable using
- `echo $SHELL`
`pwd` shows ur current directory
`uname` Gives you data about the system
- `-s` print kernel name
- `-n` print nodename or hostname
- `-r` print the release of kernel
- `-v` print version
- `-m` print machine's cpu name
- `-o` print os name
- `-a` print all of the above
`man` reads manuals
Escape sequences
- `\a` Alert
- `\b` Backspace
- `\c` Suppress trailing new line
- `\f` Form feed
- `\n` New line
- `\r` Carriage return
- `\t` Horizontal tab
- `\` break commands to more lines
Environment Variables `set` or `env` to check
- `USER` name of the logged-in user
- `PATH` List of directories to search for commands, colon separated
- `EDITOR` Default editor|
- `HISTFILE`
- `HOSTNAME` Where bash should save its history (normally `.bash_history`)
- `PS1` Prompt! Play with it
- `UID` Numeric user id of the logged-in user
- `HOME` User's home directory
- `PWD` The current working directory
- `SHELL` The name of the shell
- `$` PID of the running bash shell process
- `PPID` Id of the parent process
- `?` Exit code of the last command
Command hist
- `Up and Down Arrow` :Move in the history
`Ctrl+R` : Backward Search
`Ctrl+O` : Run the command you found with Ctrl+R
`!!` : Run the last command|
`!10` : Run command number 10
`!text` : search backward for text, and run the first found command