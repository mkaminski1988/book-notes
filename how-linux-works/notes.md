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