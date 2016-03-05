## Find the size of the terminal

The
[documentation](https://docs.python.org/3/library/os.html#querying-the-size-of-a-terminal)
suggests that
[`shutil.get_terminal_size()`](https://docs.python.org/3/library/shutil.html#shutil.get_terminal_size)
be used:

```py
>>> import shutil
>>> tsize = shutil.get_terminal_size()
>>> tsize
os.terminal_size(columns=191, lines=60)
>>> tsize.columns
191
>>> tsize.lines
60
>>>
```

`shutil.get_terminal_size()` and `os.get_terminal_size()` both require
Python 3.3+.


`shutil.get_terminal_size()` examines the environment varables `LINES` and
`COLUMNS`, and secondarily uses `os.get_terminal_size()` to find the size
of `stdout`.

If you wish to take into account `stdin`, `stdout`, `stderr`, as well as
`/dev/tty`, use something like [this](find_terminal_size.py):

```py
#!/usr/bin/python
"""Based on https://stackoverflow.com/a/566752"""

import sys
import os

from fcntl import ioctl
from termios import TIOCGWINSZ
from struct import unpack


def get_terminal_size():
    """Find the size of the terminal"""
    def ioctl_gwinsz(fdsc):
        """…"""
        try:
            packed = ioctl(fdsc, TIOCGWINSZ, bytes(4))
            rowcol = unpack('hh', packed)
        except OSError:
            # Hopefully, this is an
            #     OSError: [Errno 25] Inappropriate ioctl for device
            return
        return rowcol
    rowcol = ioctl_gwinsz(sys.stdin.fileno()) \
        or ioctl_gwinsz(sys.stdout.fileno()) \
        or ioctl_gwinsz(sys.stderr.fileno())
    if rowcol is None:
        try:
            with open(os.ctermid(), 'r') as fptr:
                rowcol = ioctl_gwinsz(fptr.fileno())
        except OSError:
            pass
    if rowcol is None:
        rowcol = (
            os.environ.get('LINES', 25),
            os.environ.get('COLUMNS', 80)
        )
    return map(int, rowcol)

if __name__ == '__main__':
    (LINES, COLUMNS) = get_terminal_size()
    # Similar to the output from resize(1)
    print('COLUMNS={}; export COLUMNS;'.format(COLUMNS))
    print('LINES={}; export LINES;'.format(LINES))

# eof
```
