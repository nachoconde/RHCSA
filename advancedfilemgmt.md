# Advanced File Management

## Linux Users

Types of users:

- Regular users: Assigned to individuals to perform their job. They have restrictions applied to them.
- Superuser: Also referred to as ''root." This is the main administrative account in the system and has full access to it.
- System users: These are user accounts usually assigned to running processes or ''daemons'' to limit their reach within the system. System users are not intended to log in to the system.

```bash
id
uid=1000(user) gid=1000(user) groups=1000(user),10(wheel) conte
xt=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

| User ID | Numeric identifier of the user in the system.
Identifiers of 1000 and above are regular users. 999 and below are reserved for system users |
| --- | --- |
| Group ID  | Numeric identifier for the principal group assigned to the user |
| SeLinux Context | SELinux context for the user. |

ID-related data is stored in  `/etc/passwd` file.  To edit `vipw`

Example:

`user:x:1000:1000:user:/home/user:/bin/bash`

- Username
- Field for the encrypted password. In this case, it shows as x because it has moved to /etc/shadow,
- UID value.
- GID value.
- Description of the account.
- Home directory assigned to the user
- Command interpreter for the user. Bash is the default interpreter

## Linux Groups

The data for groups is stored in `/etc/group` To edit we must use the `vigr`tool.

Example: `wheel:x:10:user`

- wheel: This is the name of the group.
- x: This is the group password field
- 10: This is the GID value for the group itself
- user: List of users

Types of groups

- Primary group: Assigned to the files newly created by the user
- Private group: Specific group, with the same name as the user, that is created for each user
- Supplementary group: This is another group usually created for specific purposes wheel, cdroom, etc...

## File permissions

```bash
-rw-r--r-- 12 linuxize users 12.0K Apr  28 10:10 file_name
|[-][-][-]-   [------] [---]
| |  |  | |      |       |
| |  |  | |      |       +-----------> 7. Group
| |  |  | |      +-------------------> 6. Owner
| |  |  | +--------------------------> 5. Alternate Access Method
| |  |  +----------------------------> 4. Others Permissions
| |  +-------------------------------> 3. Group Permissions
| +----------------------------------> 2. Owner Permissions
+------------------------------------> 1. File Type
```

1.

| File Type | • Directories d
• Links: l
• Special permissions to run a file as a different user or group, called setuid or setgid, will appear as s.
• A special permission so that the owner can only remove or rename the file, called the sticky bit, will appear as t |
| --- | --- |
| Permissions | • R, the read permission assigned.
• W the write permission assigned.
• X, executable permission. |
| s | If found in the user triplet, it sets the setuid bit. If found in the group triplet, it sets the setgid bit. It also means that x flag is set.
When the setuid or setgid flags are set on an executable file, the file is executed with the file’s owner and/or group privileges. |
| S | Same as s , but the x flag is not set. This flag is rarely used on files. |
| t | If found in the others triplet, it sets the sticky bit.
It also means that x flag is set. This flag is useless on files. |
| T | Same as, t but the x flag is not set. This flag is useless on files. |

### ****Changing File permissions****

Chmod command modifies access rights  and can modify permissions specified in one of two ways: symbolic or octal

```bash
chmod [OPTIONS] [ugoa…][-+=]perms…[,…] FILE...
```

- `u` - The file owner.
- `g` - The users who are members of the group.
- `o` - All other users.
- `a` - All users, identical to `ugo`.

The second set of flags (`[-+=]`), the operation flags, defines whether the permissions are to be removed, added, or set:

- `` - Removes the specified permissions.
- `+` - Adds specified permissions.
- `=` - Changes the current permissions to the specified permissions. If no permissions are given after the `=` symbol, all permissions from the specified user class are removed.

```bash
#Give read, write and execute permission to the file’s owner, read permissions to the file’s group, and no permissions to all other users:
chmod u=rwx,g=r,o= filename
```

The permission number can be a 3 or 4-digits number. When 3 digits number is used, the first digit represents the permissions of the file’s owner, the second one the file’s group, and the last one all other users.

Each write, read, and execute permissions have the following number value:

- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1
- no permissions = 0

### Default permissions

Linux assigns default permissions to a file or directory at the time of its creation.  Default permissions are calculated based on the umask (user mask) permission value subtracted from a preset initial permissions value.

`umask` is a three-digit octal value that refers to read, write, and execute permissions for owner, group, and public. The default umask value is set to 0022 for the root user and 0002 for all normal users

`umask -s`

The predefined initial permission values are 666 (rw-rw-rw-) for files and 777 (rwxrwxrwx) for directories

If you want different default permissions set on new files and directories, you will need to modify the umask.  If you want different default permissions set on new files and directories, you will need to modify the umask.

### **Special File Permissions**

- **user identifier bit** **(setuid or suid)**:  defined on binary executable files to provide non-owners and non-group members the ability to run them with the privileges of the owner or the owning group, respectively
- **group identifier bit (setgid or sgid)**  on shared directories for group collaboration
- **sticky bit:** public directories for inhibiting file erasures by non-owners

#### setuid Bit

The setuid flag is set on binary executable files at the file owner level. With this bit set, the file is executed by non-owners with the same privileges as that of the file owner

A common example is the su command that is owned by the root user.

See the underlined “s” in the owner’s permission class below:

```bash
└─$ ls -l /usr/bin/su
-rwsr-xr-x 1 root root 71912 ago 20  2021 /usr/bin/su
chmod u-s /usr/bin/su
chmod u+s /usr/bin/su
```

#### setgid Bit

Set on binary executable files at the group level. The file is executed by non-owners with the exact same privileges as that of the group members

```bash
└─$ ls -l /usr/bin/write.ul
-rwxr-sr-x 1 root tty 22760 ago 20  2021 /usr/bin/write.ul
```

When a normal user executes this command to write to the terminal of another user, the command will run as if a member of the tty group is running it, and the user is able to execute it successfully

```bash
chmod g-s /usr/bin/write
chmod g+s /usr/bin/write
```

The setgid bit can also be set on group-shared directories to allow files and subdirectories created underneath to automatically inherit the directory’s owning group. The standard behavior for new files and subdirectories is to always receive the creator’s group

```bash
groupadd -g 9999 sgrp
usermod -aG sgrp user1
chown root:sgrp /sdir
#Set the setgid bit on /sdir using the chmod command:
chmod g+s /sdir
#Add write permission to the group members on /sdir and revoke all permissions from public
chmod g+w,o-rx /sdir
touch file
```

#### sticky Bit

The sticky bit is set on public and shared writable directories to protect files and subdirectories owned by normal users from being deleted or moved by other normal users. This attribute is set on the /tmp and /var/tmp directories by default

```bash
└─$ ls -l /var/tmp -d
drwxrwxrwt 7 root root 4096 feb 23 02:09 /var/tmp
```

Notice the underlined letter “t” in other’s permission fields

```bash
chmod o-t /tmp
rm /tmp/sticky
chmod o+t /tmp
```

## File Searching

```bash
find [options] [path...] [expression]
```

- The `options` attribute controls the treatment of the symbolic links, debugging options, and optimization method.  The option `-L` (options) tells the `find` command to follow symbolic links.
- Expressions
  - `-name` option followed by the name of the file you are searching for.
  - `-iname`
  - `-type`
  - `- size`
  - `-perm` permissions
  - `-exec` Execute actions over the files
  - `-delete`
  - `-user` owner

Examples

- `find /var/log/ -name`*.temp`-delete`
- `find / -user www-data -type f  -exec chown nginx {} \;`

## Access Control Lists (ACLs)

These permissions are in addition to the standard ugo/rwx permissions and the setuid, setgid, and sticky bit settings

ACLs are categorized into two groups based on their type and are referred to as access ACLs and default ACLs

- Access ACLs are set on individual files and directories
- default ACLs can only be applied at the directory level with files and subdirectories inheriting them automatically

Two commands to view and manage ACLs on files and directories

- getfacl
- setfacl

```bash
[root@wildfly01 ~]# getfacl nacho-test/
# file: nacho-test/
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
```

| u[ser]:UID:perms | Permissions assigned to a named user.
If this field is left blank, the permission will be the owner of the file or directory, which is equivalet using the chmod command. |
| --- | --- |
| g[roup]:GID:perms | Permissions assigned to a named group (group GID) |
| o[ther]:perms | Permissions assigned to users that are neither nor part of the owning group |
| m[ask]:perms | determines the maximum allowable permissions placed for a named user or group on a file or directory. If it is set to rw, for instance, no named user or group will exceed those permissions. |

Examples

```bash
setfacl -m m:rw 
getfacl nacho-test/
# file: nacho-test/
# owner: root
# group: root
user::rwx
group::r-x                      #effective:r--
mask::rw-
other::r-x

setfacl -m u:wildfly:6 nacho-test/
#ACL indicated with +
ls -la nacho-test
drwxrw-r-x+ 2 root root   6 Feb 28 18:25 .

 getfacl nacho-test/
# file: nacho-test/
# owner: root
# group: root
user::rwx
user:wildfly:rw-
group::r--
mask::rw-
other::r-x

#delete
setfacl -xu:wildfly nacho-test
```

Default ACLs:

```bash
setfacl -dm u:user1:7,u:user2:6 carpeta

setfacl -dm u:wildfly:6 nacho-test
getfacl nacho-test/
# file: nacho-test/
# owner: root
# group: root
user::rwx
group::r--
mask::r--
other::r-x
default:user::rwx
default:user:wildfly:rw-
default:group::r--
default:mask::rw-

mkdir directoriosub
getfacl directoriosub/

# file: directoriosub/
# owner: root
# group: root
user::rwx
user:wildfly:rw-
group::r--
mask::rw-
other::r-x
default:user::rwx
default:user:wildfly:rw-
default:group::r--
default:mask::rw-
default:other::r-x
```
