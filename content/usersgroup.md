# User and Groups User Management

## Users

A user account is used to provide security boundaries between different people and programs that can run commands.

Users have user names to identify them to human users and make them easier to work with. Internally, the system distinguishes user accounts by the unique identification number assigned to them, the user ID or UID.

If a user account is used by humans, it will generally be assigned a secret password that the user will use to prove that they are the actual authorized user when logging in.

Three main types of user accounts:

1. Superuser account is for administration of the system. The name of the superuser is root and the account has UID 0.
2. System has system user accounts which are used by processes that provide supporting services. These processes, or daemons, usually do not need to run as the superuser. They are assigned non-privileged accounts that allow them to secure their files and other resources from each other and from regular users on the system.
3. Regular user accounts

### Commands to manage users activity

- `WHO`:  displays the login name of the user and the terminal session device filename (pts stands for pseudo terminal session, and tty identifies a terminal window on the console)
- `WHAT`:  displays information in a similar format as the who command, but it also tells the length of time the user has been idle for (IDLE), along with the CPU time used by all processes including any existing background jobs attached to this terminal (JCPU), the CPU time used by the current process (PCPU), and current activity (WHAT).
- `LAST`: reports the history of successful user login attempts and system reboots by consulting the wtmp file located in the /var/log directory
- `LASTB` reports the history of unsuccessful user login attempts by reading the btmp file located in the /var/log directory
- `LASTLOG`:   reports the most recent login evidence information for every user account that exists on the system. This information is captured in the lastlog file located in the /var/log directory
- `ID`: displays the calling user’s UID (User IDentifier), username, GID (Group IDentifier), group name, all secondary groups the user is a member of, and SELinux security context

## Authentication Files

User account information for local users is stored in four files:

1. passwd
2. shadow
3. group
4. gshadow

### 1. passwd

| user1 | x | 1 | 1 | user1 | /usr/sbin | /usr/sbin/nologin |
| --- | --- | --- | --- | --- | --- | --- |
| USER | PASSWORD | UID | GID | USER INFO | HOME | SHELL |

### 2. shadow File

Passwords are hashed and stored in a more secure file /etc/shadow.

There are certain limits on user passwords in terms of expiration, warning period, etc., that can also be applied on a per-user basis. These limits and other settings are defined in the /etc/login.defs file, which the shadow password mechanism enforces on user accounts

```bash
mark:$6$.n.:17736:0:99999:7:::
[--] [----] [---] - [---] ----
|      |      |   |   |   |||+-----------> 9. Unused
|      |      |   |   |   ||+------------> 8. Expiration date
|      |      |   |   |   |+-------------> 7. Inactivity period
|      |      |   |   |   +--------------> 6. Warning period
|      |      |   |   +------------------> 5. Maximum password age
|      |      |   +----------------------> 4. Minimum password age
|      |      +--------------------------> 3. Last password change
|      +---------------------------------> 2. Encrypted Password
+----------------------------------------> 1. Username
```

1. Username of the account this password belongs to.
2. The encrypted password of the user.
3. The day on which the password was last changed.
4. The minimum number of days that have to elapse since the last password change before the user can change it again.
5. The maximum number of days that can pass without a password change before the password expires.
6. An empty field means it does not expire based on time since the last change.
7. Warning period. The user will be warned about an expiring password when they login for this number of days before the deadline.
8. Inactivity period. Once the password has expired, it will still be accepted for login for this many days. After this period has elapsed, the account will be locked.
9. The day on which the password expires.

### 3. Group File

Each row in the file stores information for one group entry

Can contain a hashed password, which may be set with the gpasswd command for non-group members to access the group temporarily using the newgrp command.

### 4. Gshadow File

The shadow password implementation also provides an added layer of protection at the group level. With this mechanism in place, the group passwords are hashed and stored in a more secure file /etc/gshadow

## Useradd / UserMod / UserDel

`useradd` command picks up the default values from the /etc/default/useradd and /etc/login.defs

Both files store defaults including those that affect the password length and password lifecycle. Including:

- Starting GID (GROUP)
- Home directory location (HOME)
- Number of inactivity days between password expiry and permanent account disablement
(INACTIVE)
- Account expiry date (EXPIRE)
- Login shell (SHELL)
- Skeleton directory location to copy user initialization files from (SKEL)
- and whether to create mail spool directory (CREATE_MAIL_SPOOL).

Tools:

- `useradd` to add a new user to the system,
- `usermod` to modify the attributes of an existing user,
- `userdel` to remove a user from the system.
- `passwd` command is available to set or modify a user’s password.

`UserAdd` options

- -b (--base-dir) Defines the absolute path to the base directory
- -c (--comment)
- -d (--home-dir) Defines the absolute path to the user home
- -D (--defaults) Displays the default settings from the /etc/default/useradd file and modifies them.
- -e (--expiredate) Specifies a date on which a user account is automatically disabled. The format for the da specification is YYYY-MM-DD.
- -f (--inactive) Denotes maximum days of inactivity between password expiry and permanent account disablement.
- -g (--gid) Specifies the primary GID
- -G (--groups) Specifies the membership
- -k (--skel) Specifies the location of the skeleton director (default is /etc/skel), which stores default use files.
- -m (--create-home) Creates a home directory
- -o (--non-unique) Creates a user account sharing the UID of an existing user.
- -r (--system) Creates a service account with a UID below and a never-expiring password.
- -s (--shell) Defines the absolute path to the shell file
- -u (--uid) Indicates a unique UID.

```bash
usermod --lock user1
usermod --unlock user1
usermod --expiredate 2022-01-01 user1
usermode -e "" jane #never expires
sudo usermod -a -G developers jane # add user to the group
```

### No-Login (Non-Interactive) User Account

The nologin shell is a special purpose program that can be employed for user accounts that do not require login access to the system.

 It is located in the /usr/sbin (or /sbin) directory.

With this shell assigned, the user is refused. Typical examples of user accounts that do not require login access are the service accounts such as ftp, apache, and sshd.

### Owning User and Owning Group

Every file and directory has an owner.

- By default, the creator assumes the ownership
- Every user is a member of one or more groups.
- By default, the owner’s group is assigned to a file or directory.

`chown y chgrp`

## Super User

Linux provides normal users the ability to run a set of privileged commands or to access non-owning files without the knowledge of the root password, using `sudo` command.

The `su` command allows users to switch to a different user account. If you run su from a regular user account, you will be prompted for the password of the account to which you want to switch.

```bash
#Switch from user1 into root without executing the startup scripts
su
#Switch to root executing startup scripts
su -
#Switch to user1 executing startup scripts
su - user1
```

The su command with the -c option, it can be used like the Windows utility runas to run an arbitrary program as another user.

### SUDO

In some cases, the root user's account may not have a valid password at all for security reasons. In this case, users cannot log in to the system as root directly with a password, and su cannot be used to get an interactive shell. One tool that can be used to get root access in this case is `sudo`.

`sudo -i`  to gain a shell

To configure sudo we use /etc/sudoers file. To edit `visudo`, which creates a copy of the file as sudoers.tmp and applies the changes there.

Following file enables sudo access for members of group wheel.

```bash
%wheel ALL=(ALL) ALL
```

- %wheel is the user or group to whom the rule applies. A % specifies that this is a group
- The ALL=(ALL) specifies that on any host that might have this file, wheel can run any command.
- The final ALL specifies that wheel can run those commands as any user on the system.

/etc/sudoers also includes the contents of any files in the /etc/sudoers.d directory as part of the configuration file

- Using supplementary files under the /etc/sudoers.d directory is convenient and simple. You can enable or disable sudo access simply by copying a file into the directory or removing it from the directory.

Example:

- To enable full sudo access for the user user01, you could create /etc/sudoers.d/user01 with the following content:

```bash
user01 ALL=(ALL) ALL
```

To allow an user to run commands without entering password:

```bash
ansible ALL=(ALL) NOPASSWD:ALL
```

We can restrict access to some commands

```bash
user1 ALL=/usr/bin/cat
%dba ALL=/usr/bin/cat
```

In another way

```bash
Cmnd_Alias PKGCMD = /usr/bin/yum, /usr/bin/rpm
User_Alias PKGADM = user1, user100, user200
PKGADM ALL = PKGCMD
```

## Password Aging

These controls are applied to all user accounts at the time of their creation and can be set explicitly on a per-user basis later.

Password aging information is stored in the /etc/shadow file and its default policies in the /etc/login.defs.

`chage`:  used to set or alter password aging parameters on a user account

- -d (--lastday) set date of last password change
- -E (--expiredate) Sets an explicit date the number of days since the UNIX time on w
user account is deactivated.
- -I (--inactive) Defines the number of days of inactivity after password expiry and before the account is locket
- -l Lists password aging attributes set on a user account.
- -m (--mindays) Indicates the minimum number of days elapse before the password can be changed
- -M (--maxdays) Denotes the maximum number of days of validity before the user password expires and can be changed. .
- -W (--warndays)

```bash
#Mark the password for jane as expired to force her to immediately change it the next time she logs in.
sudo chage --lastday 0 jane
```

`passwd`  set or modify a user’s password; however, you can also use this command to modify the password aging attributes and lock or unlock their account

- -d  —delete
- e (--expire) Forces a user to change their password upon logon.
- -i (--inactive) Defines the number of days of inactivity
- -l (--lock) Locks a user account.
- -n (--minimum)
- -S (--status) Displays the status information for a user.
- -u (--unlock) Unlocks a locked user account.
- -w (--warning) Designates the number of days for which the gets alerts to change their password before
- -x (--maximum) Denotes the maximum number of days

`usermod`

- L (--lock) Locks a user account by placing a mark (!)
- -U (--unlock) Unlocks a user’s account by removing the exclamation mark

## Linux Groups

Group information is stored in

- /etc/group file and the
- /etc/login.defs  default policies
- /etc/gshadow file stores group administrator information and group-level passwords.

`groupadd` command command uses the next available GID from the range specified in the /etc/login.defs

- g (--gid) Specifies the GID to be assigned
- -o (--non-unique) Creates a group with a matching GID and another group
- -r System group GID bellow 1000?
- groupname

`groupmod` Modify a group

`groupdel` Delete a group

```bash
groupadd -g 5000 gr
groupadd -o -g 5000 dba 
```
