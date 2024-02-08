## Useful Linux Shell Commands

*NOTE: commands with one (1) dash "-" tend to be one letter, while those with two (2) dashes "--" use more verbose language. Also, the short commands can be combined into one.*

All man pages can be found here: https://man.archlinux.org/man/

---

### File and Directory Manipulation

- `ls` - list directory contents
  - `-a` or `--all` : list files prefixed with ".", which are usually hidden files
  - `-d` or `--directory` : list directories
  - `-l` : list in long format, to include permisions, owner, group, size, & date
  - `-la` : list in long format, and include hidden files
  - https://man.archlinux.org/man/ls.1.en
- `mkdir` - make a new directory
  - `-p` : create parent directories as needed. ex: `mkdir -p /test/subtest` will create the /test if it doesn't already exist
  - create more than one directory at time by spacing arguments ex: `mkdir test test1 test2` will create three folders at once
  - https://man.archlinux.org/man/mkdir.1
- `>` - Push to file
  - ex: `> test.txt` will create an empty test.txt file

### Printing to Screen and to Files

- `echo` - display text directly to the terminal
  - ex: `echo hello` produces `hello` on the next line of the terminal
  - `>` : `echo` can be combined with `>` like `echo hello > test.txt` to pass a string to a file. THIS WILL OVERWRITE THE ENTIRE FILE!
  - `>>` : use this instead of `>` if you only want to add to the end of the file. ex: `echo hello >> test.txt` will add an extra line with "hello" at the bottom
  - https://man.archlinux.org/man/echo.1
- `cat` - concatenate files and print on the standard output (similar to `echo`, but from a file)
  - ex: `cat test.txt` would produce `hello` on the next terminal line if test.txt had one line with "hello" as the text.
  - `>` : Push a file into another file. ex: `cat test.txt > test2.txt` will make text2.txt have the exact same contents as text.txt
  - `>>` : Add onto another file. ex: `cat test.txt >> test2.txt` will add test.txt to the bottom of test2.txt
