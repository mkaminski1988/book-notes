# How Linux Works

## Chapter 1

* The term **image** can refer to a particular arrangement of bits.
* The MMU (Memory Management Unit) is a hardware translator that maps process 
memory addresses to physical memory addresses.
* Linux system call names are typically denoted with parenthesis. (ex: `fork()` and `exec()`)
* Other than *init*, all Linux processes start as a result of a `fork()` followed by `exec()`.
* The kernel does not manage *usernames*. It deals with *userids* (*usernames* map to *userids*).

## Chapter 2

* All Unix shells derive from the *Bourne shell* originally developed for Unix.
Bash is a derivative of this shell.
* The question mark `(?)` glob character instructs the shell to match just one
character.
* To escape globbing, surround text with quotes.
* `grep -v` inverts seatch.
* Use `?search` to search backwards in `less` command.
* Once a match is found in `less` output, press `n` to continue searching.
* `diff -u` shows a unified diff output.
* `file` displays the type of a file's contents.
* `-print` is the default action of the `find` command. Some old 
implementations had no default required an explicit `-print` action.
* Shell variables and environment variables are similar. The difference is that
shell variables are not passed to all progams that the shell runs. To pass a shell
variable to a program, you must `export` it.
* Some command-line keystrokes:
    * `CTRL-W` - Erase previous word.
    * `CTRL-U` - Erase from cursor to beginning of line (in `zsh`, this erases
    entire line).
    * `CTRL-Y` - Paste erased text.
* The number that appears next to a `man` page name is the section. Section 1
(`man(1)`) is the user commands section. Section 2 (`man(2)`) is the system
calls section.
* "Clobber" means to overwrite a file.
* `&>` redirects both `stdout` and `stdrr`. (`&>word` and `>&word` are the
same, the first is preferred.)
* `$$` is a shell variable that evaluates to the current shell’s PID.
* `CTRL-Z` suspends a process. The `fg` command returns back to the suspended process.
* `fg` can also be used to return to a program running in the background using `cmd &`.
* File permissions
    * The file mode contains several parts. Given `-rw-r--r--`:
        * The first dash (`-`) indicates a file. Directors are denoted with `d`.
        * The first segment is `rw-`, which is for user permissions.
        * The second segment is `r--`, which is for group permissions.
        * The third segment is `r--`, which is for other (or "world") permissions.
    * Each permission set can contain 4 different representations:
        * `r` is readable.
        * `w` is writable.
        * `x` is executable.
        * `-` means nothing (just a placeholder).
    * You can set group and other permissions with `chmod`.
        * `chmod g+r file` adds group read permisison.
        * `chmod o+r file` adds world read permisison.
        * `chmod g-r file` removes group read permisison.
        * `chmod go+r` adds group and world read permission in one shot.
    * Directories must be set as executable in order to access files within them.
    * `umask` sets default permissions for any new files created by a user.
    This is set in one of the startup files. `022` allows all users to see new
    files.
* File archiving and compression
    * `tar [x,c,t]vf file.tar`
        * `x`: extract, `c`: compress, `t`: display table of contents
        * `v` vebose
        * `f` filename (must be followed by filename)
    * For tar, you can compress/uncompress by using `z` option.
    * `gzip file`: compress, `gunzip file.gz` uncompress
    * `*.Z` files were compressed with legacy Unix compression program. gzip 
    can read them but not create.