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

## Chapter 3

* The udev system enables user-space programs to automatically configure and
use new devices.
* `udev` manages device files located in `/dev`.
* Device type can be determined with `ls -l`.
    * `b`: block - Access data in fixed chunks.
    * `c`: character - Character streams, can't do random access.
    * `p`: pipe  - Like character device, but with another process at end of 
    stream instead of kernel driver.
    * `s`: socket - Unix domain sockets used for IPC.
* Not all devices have device files because they may not work with the
block/character device pattern, such as network cards.
* The `/sys/devices` device path provides more detailed device information than the
`/dev` directory.
* The `dd` command copies data in blocks of fixed size.
* Linux has two primary display modes: `text mode` and `X Window Server`.
    * In text mode, you can switch between consoles using `ALT+F#`.
    * In graphics mode, you can switch between consoles using `CTLR+ALT+F#`.
    * `/dev/tty*` contains virtual consoles, which are Kernel-emulated terminals.
    * `/dev/pts/*` contains psuedoterminal devices.
    * `/dev/tty` is the terminal of the current process.
* `devtmpfs` is a temporary file system that stores device information at boot
time and is eventually mounted to `/dev` upon completion. No user-space support
is required to have a working `/dev` directory during boot time.
* `udevd` handles all user space events raised when hardware devices are added
into the system or removed from it.
* Each device has a major-minor pair. The major number refers to the driver
(floppy disk, hard drive, etc) and the minor refers to something specific, such
as the bus the device is connected to.
* The SCSI protocol is a big-endian peer-to-peer packet based protocol.
* SCSI commands can be sent over different kinds of busses, such as SATA, SAS, ATAPI, etc.
* From Wikipedia: ATA Packet Interface (ATAPI) is a protocol that has been added to Parallel ATA and Serial ATA so that
a greater variety of devices can be connected to a computer than with the ATA command set alone. It carries SCSI
commands and responses through the ATA interface. ATAPI devices include CD-ROM and DVD-ROM drives, tape drives,
magneto-optical drives, and large-capacity floppy drives such as the Zip drive and SuperDisk drive.

## Chapter 4

* Partitions are defined in a partition table. Master Boot Record (MBR) is the traditional type, although a new type
called Globally Unique Identifier Partition Table (GPT) is starting to gain traction.
* There are several types of paritioning tools:
    * **parted**: text-based tool that supported MBR and GPT
    * **gparted**: gui-based version of parted
    * **fdisk**: traditional tool that does not support GPT
    * **gdisk**: version of fdisk that supports GPT but not MBR
* There are several MBR partition types:
    * **primary**: a normal subdivision of a disk, normally limited to 4
    * **logical**: one of the primary partitions that contain extended partitions
    * **extended**: multiple partitions inside logical partition used by OS
* `CHS` geometry is an early method for giving addresses to blocks of data: cylinder, head, sector.
    * **cylinder**: concentric circles on multiple platters that contain data
    * **head**: reads data from cylinders
    * **sector**: subdivisions of cylinders
* **Logical Block Addressing (LBA)** is an hardware scheme for addressing locations on disk by block number.
* The typical CHS data reported by OS is incorrect. Everything depends on LBA.
* CHS is still relevant for determing partition boundaries, although LBA is used to determine the precise locations.
* For SSDs, you want to make sure partitions are correctly aligned on a 4096-byte boundaries.
* Filesystems are basically databases for transforming block devices into a sophisticated file hierarchy.
* Because filesystem names are dependent on the order in which they are discovered by the kernel, you can depend on
UUIDs to consistently identify filesystems. UUIDs are the preferred method for mounting devices in `/mnt/fstab`.
* File system tables:
    * `fstab` contains a list of devices to mount at boot time
    * `mtab` contains a list of devices currently mounted
* Mount options:
    * Short options
        * `-r`: read-only
        * `-n`: don't update runtime database (`fstab`)
        * `-t`: filesystem type
    * Long options (denoted by `-o`)
        * `exec,noexec`: enables/disables execution of programs on filesystem
        * `suid,nosuid`: enables/disables setuid programs (allows users to execute program with temporarily elevated
        permissions)
        * `ro`: mount filesystem in read-only mode
        * `rw`: mount filesystem in read-write mode
        * `conv=rule`: for FAT systems, sets newline conversion rule.
            * `binary`: disable character translation
            * `text`: treat all files as text
            * `auto`: convert files based on file extension (i.e. txt)
* `/etc/fstab` mount options:
    * `defaults`: mount defaults: read-write mode, executable files enabled, setuid bit enabled 
    * `errors`: (ext2-specific) behavior for when system has trouble mounting filesystem
    * `noauto`: prevents boot-time mount, such as for floppy disks and CD-ROMs
    * `user`: allows non-privileged users to run mount on device
* `/etc/fstab.d` directory and systemd units are alternatives to `/etc/fstab` that are gaining traction.
* When running `df` as a non-root user, the sum of `Used` and `Available` blocks does not add up to the total number of
blocks because a certain percentage is made up of hidden `reserved` blocks. This helps prevent non-root users from
filling up the disk.
* Do not run `fsck` on a mounted filesystem. Exception: you can mount the root partition in read-only single user mode.
* `fsck` places disconnected inodes in the `lost+found` directory. The user can guess the filename based upon the file
contents.
* Filesystems are not just for representing files on storage media. They can also serve as system interfaces for
    inspecting kernel resources, processes, etc.
    * `proc`: Mounted on `/proc`, allows inspection of processes and kernel/hardware information.
    * `sysfs`: Mounted on `/sys/devices`, used to view information about devices and manage them.
    * `tmpfs`: Mounted on `/run` and elsewhere, allows users to use physical memory and swap as temporary storage.
* An `inode` is a set of data that describes a file's type, permissions, and data location.
* An `inode` **link count** is the number of total directory entries that point to an inode.
* A hard link is just a manually created entry in a directory to an inode that already exists.
* A **block bitmap** is a datastruct used to quickly tell whether a block is in use or not. Problems occur when there
is a mismatch between the block bitmap and inode data table.