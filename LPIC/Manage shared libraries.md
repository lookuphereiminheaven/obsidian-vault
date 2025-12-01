`ldd` to see if a program is statically or dynamically linked
When a program needs a shared library, the system will search files in this order
- LD_LIBRARY_PATH environment variable
- Programs PATH
- `/etc/ld.so.conf` (Which might load more files from `/etc/ld.so.conf.d/` in its beginning or its end)
- `/lib/`, `/lib64/`, `/usr/lib/`, `/usr/lib64/`
