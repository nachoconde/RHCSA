# Basic File management

RHEL supports seven types of files:

- Regular: contain text or binary data, may be shell scripts or commands in the binary form. When you list a directory, all line entries for files in the output that begin with the hyphen character (-) represent regular files.
- Directory: The letter “d” at the beginning of each line entry identifies the file as a directory
- Block and Character Special Device Files: /dev directory that is used by the system to communicate with that device. There are two types of device files: character (or raw) and block.  Every hardware device such as a disk, CD/DVD, printer, and terminal has an associated device driver loaded in the kernel. The kernel communicates with hardware devices through their respective device drivers. Each device driver is assigned a unique number called the major number, which the kernel uses to recognize its type.
- Symbolic link: A symbolic link (a.k.a. a soft link or a symlink) may be considered a shortcut to another file or directory. The line entry will begin with the letter “l”, and an arrow will be pointing to the target link
- Named pipe
- Socket

## Compression and Archiving

Tools:

- Tar
- Star
- gzip
- bzip2

The tar and star commands have the ability to preserve general file attributes such as ownership, owning group, and timestamp as well as extended attributes such as ACLs and SELinux contexts

### TAR

- -c: Create
- -x: Extract
- -f: File
- -t: List
- -z: Compresses a tarball with gzip
- -j Compresses a tarball with bzip2

```bash
tar -cf etc-backup.tar /etc
tar -xf etc-backup.tar
```

### GZIP / GUNZIP

- -r to compress an entire directory tree,
- -l to display compression information about a gzipped file

### BIZIP2 / BUNZIP2

The function of both gzip and bzip2 is the same: to compress and decompress
files. However, in terms of the compression and decompression rate, bzip2
has a better compression ratio (smaller target file size), but it is slowe

## File Editing - VIM

All text editing within vim takes place in a buffer (a small chunk of memory used to hold file updates). Changes can either be written to the disk or discarded.

3 modes of operation

1. Command mode: Default one
2. Input mode, i to enter Esc to return to command mode
3. Visual Mode: v

| Command | Action |
| --- | --- |
| u | undo |
| y | yank |
| P | Put |
| a | Append text after current position |
| A | Append text end of current line |
| i | text before the current cursor |
| I | Text before current position |
| x | Deletes character at cursor |
| X | Deletes character before the cursor |
| dw | delete world |
| /string | Buscar |
| :%S/old/new | Sustituir |

## Links

A file’s metadata includes several pieces of information, such as the file type, size, permissions, owner’s name, owning group name, last access/modification times, link count, number of allocated blocks, and pointers to the data storage location

This metadata takes 128 bytes of space for each file. This tiny storage space is referred to as the file’s inode (index node).

- An inode is assigned a unique numeric identifier that is used by the kernel for accessing, tracking, and managing the file.
- In order to access the inode and the data it points to, a filename is assigned to recognize it and access it. This mapping between an inode and a filename is referred to as a link

Linking files or directories creates additional instances of them, but all of them eventually point to the same physical data location in the directory tree.

Linked files may or may not have identical inode numbers and metadata depending on how they are linked.

Links are created between files or between directories, but not between a file and a directory.

**Hard Link**

A hard link is a direct reference to a file via its inode

- One or more filenames and an one inode. This implies that all hard-linked files will have identical metadata.
- Cannot cross a file system boundary
- Cannot be used to link directories

```bash
ln file10 file20
ls -li
```

**Soft Link**

Symbolic links are essentially shortcuts that reference to a file instead of its inode value.

- Possible to associate one file with another. The concept is analogous to that of a shortcut in Windows
- Can access the file directly via the actual filename as well as any of the shortcut
- can cross a file system boundary
- it can be used to link directories

    ```bash
     ln -s (s for symbolic)
    ```

**Differences between Copying and Linking**

| Copying | Linking |
| --- | --- |
| Duplicate of source file | Shortcut that points to the filename |
| Own data at unique location | Same data |
| Unique inode | - Hard link: share inode

- Softlink: Each link has its own inode number, that points to the source |
| If erased lose file | -Hard link: if erased the link, file untouched
-Softlink: no impact if deleted |
| Permissions idenpendts | Permissions managed on the source file |
