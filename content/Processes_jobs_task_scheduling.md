# Processes, jobs and task scheduling

## PROCESSES

A process is a running instance of a launched, executable program that Consists of:

- An address space of allocated memory
- Security properties including ownership credentials and privileges
- One or more execution threads of program code
- Process state

The environment of a process includes:

- Local and global variables
- A current scheduling context
- Allocated system resources, such as file descriptors and network ports

An existing (parent) process duplicates its own address space (fork) to create a new (child) process structure.

Every new process is assigned a unique process ID (PID) for tracking and security. The PID and the parent's process ID (PPID) are elements of the new process environment. Any process may create a child process. All processes are descendants of the first system process, systemd.

Through the fork routine, a child process inherits security identities, previous and current file descriptors, port and resource privileges, environment variables, and program code. A child process may then exec its own program code. Normally, a parent process sleeps while the child process runs, setting a request (wait) to be signaled when the child completes. Upon exit, the child process has already closed or discarded its resources and environment. The only remaining resource, called a zombie, is an entry in the process table. The parent, signaled awake when the child exited, cleans the process table of the child's entry, thus freeing the last resource of the child process. The parent process then continues with its own program code execution.

### Process states

Processes are assigned a state, which changes as circumstances dictate.

| NAME | FLAG | DESCRIPTION |
| --- | --- | --- |
| Running | R | Executing on a CPU or waiting to run. |
| Sleeping | S | INTERRUPTIBLE: Waiting for some condition: a hardware request, system resource access, or signal. |
|  | D | UNINTERRUPTIBLE: does not respond to signals |
|  | K | KILLABLE allow a waiting task to respond to the signal that it should be killed |
|  | I | IDLE: A subset of state D. The kernel does not count these processes when calculating load average |
| Stopped | S | STOPPED: The process has been Stopped (suspended), usually by being signaled by a user or another process |
| Zombie | Z | A child process signals its parent as it exits |
|  | D | When the parent cleans up (reaps) the remaining child process structure, the process is now released completely |

Information for each running process is recorded and maintained in the /proc file system

### ps command

The `ps` command offers plentiful switches that influence its output, whereas top is used for real-time viewing and monitoring of processes and system resources.

Output returns the elementary information about processes in four columns. These processes are tied to the current terminal window.

- PID
- Terminal (TTY) the process spawned in
- The cumulative time (TIME) the system CPU has given to the process,
- Name of the actual command or program (CMD) being executed

Options

The Linux version of ps supports three option formats:

- UNIX (POSIX) options, which may be grouped and must be preceded by a dash
- BSD options, which may be grouped and must not be used with a dash
- GNU long options, which are preceded by two dashes

For example, ps -aux is not the same as ps aux.

- ps with no options selects all processes with the same effective user ID

| Options | Meaning |
| --- | --- |
| ps aux | Displays all processes including processes without a controlling terminal |
| ps lax | Long listing provides more technical detail, but may display faster by avoiding user name lookups. |
| ps -ef | Display all processes. |
| ps -U user1
Ps -G root | Specific users and groups |

### top command

The top program is a dynamic view of the system's processes, displaying a summary header followed by a process or thread list similar to ps information. Unlike the static ps output, top continuously refreshes at a configurable interval, and provides capabilities for column reordering, sorting, and highlighting. User configurations can be saved and made persistent.

Output

- The process ID (PID)
- User name (USER) is the process owner.
- Virtual memory (VIRT) is all memory the process is using, including the resident set, shared libraries, and any mapped or swapped memory pages. (Labeled VSZ in the ps command.)
- Resident memory (RES) is the physical memory used by the process, including any resident shared objects. (Labeled RSS in the ps command.)
- Process state (S) displays as:
  - D = Uninterruptible Sleeping
  - R = Running or Runnable
  - S = Sleeping
  - T = Stopped or Traced
  - Z = Zombie

### Listing a Specific Process

To list only the PID of a specific process

- `pidof`
- `pgrep`

### Understanding Process Niceness and Priority

The process scheduler on the system performs rapid switching of processes, giving the notion of concurrent execution of multiple processes.

A process is spawned at a certain priority, which is established at initiation based on a numeric value called niceness (a.k.a. a nice value).

- There are 40 niceness values, with -20 being the highest or the most favorable to the process, and +19 being the lowest or the least favorable to the process
- A higher niceness lowers the execution priority of a process
- A lower niceness increases it.
- A process running at a higher priority gets more CPU attention.
- A child process inherits the niceness of its calling process in its priority calculation.
- A normal user can only make your processes nicer,
- A root user can raise or lower the niceness of any process.

RHEL provides the

- `nice` command to launch a program at a non-default priority
- `renice` command to alter the priority of a running program

The ps command with the -l option along with the -ef options can be used to list priority (PRI, column 7) and niceness (NI, column 8) for all processes:

```bash
F S UID          PID    PPID  C PRI  NI ADDR SZ WCHAN    RSS PSR STIME TTY          TIME CMD
4 S root           1       0  0  80   0 - 41140 -       8428   1 Mar06 ?        00:00:11 /sbin/init
1 S root           2       0  0  80   0 -     0 -          0   0 Mar06 ?        00:00:00 [kthreadd]
1 I root           3       2  0  60 -20 -     0 -          0   0 Mar06 ?        00:00:00 [rcu_gp]
1 I root           4       2  0  60 -20 -     0 -          0   1 Mar06 ?        00:00:00 [rcu_par_gp]
1 I root           6       2  0  60 -20 -     0 -          0   0 Mar06 ?        00:00:00 [kworker/0:0H-events_highpri]
```

### Controlling Processes with Signals

Each signal is associated with a unique numeric identifier, a name, and an action.

A list of available signals can be viewed with the kill command using the -l option:

| NUMBER  | SHORT NAME | PURPOSE |
| --- | --- | --- |
| 1 | HUP | Used to report termination of the controlling process of a termina |
| 2 | INT | Causes program termination. Can be blocked or handled. Sent by pressing INTR key combination |
| 3 | QUIT | Similar to SIGINT, but also produces a process dump at termination. Sent by pressing QUIT key combination (Ctrl+\ |
| 9 | KILL | Causes abrupt program termination. Cannot be blocked, ignored, or handled; always fatal. |
| 15 | TERM | Unlike SIGKILL, can be blocked, ignored, or handled. The “polite” way to ask a program to terminate; allows self-cleanup |
| 18 | CONT | Sent to a process to resume, if stopped. Cannot be blocked. |
| 19 | STOP | Suspends the process. Cannot be blocked or handled. |
| 20 | TSTP | Unlike SIGSTOP, can be blocked, ignored, or handled. Sent by pressing SUSP key combination (Ctrl+Z) |

Commands to pass the signal to a process:

- `kill`: Despite its name, the kill command can be used for sending any signal. You can use the kill -l command to list the names and numbers of all available signals.
- `killall` can signal multiple processes, based on their command name.
- `pkill`  send a signal to one or more processes which match selection criteria

## JOBS

Job control is a feature of the shell which allows a single shell instance to run and manage multiple commands.

- `sleep 1000 &` Running jobs in backgroud
- `Jobs` display jobs
- `bg` move jobs to background
- `fg %1` moves job 1 to foregound
- `ctrl+z` suspend request

## Job Scheduling

Job scheduling allows a user to submit a command for execution at a specified time in the future.

Job scheduling and execution is taken care of by two service daemons:

- `atd` manages the jobs scheduled to run one time in the future.  The atd daemon retries a missed job at the same time next day
- `crond`: is responsible for running jobs repetitively at pre-specified times. At startup, this daemon reads the schedules in files located in the `/var/spool/cron` and `/etc/cron.d` directories, and loads them in the memory for on-time execution. It scans these files at short intervals and updates the inmemory schedules to reflect any modifications.

### Controlling User Access

All users are allowed to schedule jobs using the at and cron services. However, this access may be controlled and restricted to specific users only.

- at.allow
- at.deny
- cron.allow
- cron.deny

Each file takes one username per line. The root user is always permitted; it is affected neither by the existence or non-existence of these files, nor by the inclusion or exclusion of its entry in these files

| at.allow/ cron.allow | at.deny/cron.deny | Impact |
| --- | --- | --- |
| Exits and contains usrs | Existence does not matter | Permitted |
| Exits, empty | Existence does not matter | No users are permitted |
| No exist | Exits with users | All users, permitted |
| No exist | Exits, empty | All users, permitted |
| No exist | No exist | No users are permitted |

By default, the deny files exist and are empty, and the allow files are nonexistent. This opens up full access to using both tools for all users

### Scheduler Log File

All activities for atd and crond services are logged to the `/var/log/cron` file

The file also keeps track of other events for the crond service such as the service start time and any delays.

### Using at

The at command is used to schedule a one-time execution of a program in the future. All submitted jobs are spooled in the /var/spool/at directory and executed by the atd daemon at the specified time. Each submitted job will have a file created containing the settings for establishing the user’s shell environment to ensure a successful execution

Examples

```bash
at -f /home/backuk.sh 'now + 2 hours'
at 11:30pm 3/7/22
Ctrl+d at the second at> prompt to complete the job submission
```

Time examples

```bash
now +5min
teatime tomorrow (teatime is 16:00)
noon +4 days
5pm august 3 2021
```

Commands

| at -d 5 |  |
| --- | --- |
| atrm [ID] | Remove task |
| atq | get an overview of the pending jobs |
| at -c JOBNUMBER | Inspect |

### Crontab

Crond executes cron jobs on a regular basis if they comply with the format defined in the `/etc/crontab` file.

Crontables are located in

- `/var/spool/cron` directory. Each authorized user with a scheduled job has a file matching their login name in this directory, the crontable for user1 would be `/var/spool/cron/user1`
- `/etc/crontab` file
- `/etc/cron.d` directory
- The crontab system includes repositories for scripts that need to run every hour, day, week, and month. These repositories are directories called `/etc/cron.hourly/, /etc/ cron.daily/, /etc/cron.weekly/, and /etc/cron.monthly/`

The crond daemon scans entries in the files at the three locations to determine job execution schedules

Commands

| crontab -l | List |
| --- | --- |
| crontab -e | Edit |
| crontab -r | Remove jobs |
| crontab filename | Remove all jobs, and replace with the jobs read from filename |
| crontab -u | For users who wish to modify a different user’s crontable |

The `/etc/crontab` file specifies the syntax that each user cron job must comply with in order for crond to interpret and execute it successfully

Fields:

1. Minutes
2. Hours
3. Day of the month
4. Month
5. Day of the week: 0 Sunday
6. Command

Example: `15 12 15 * Fri command ->`  the 15th of every month, and every Friday at 12:15. When the Day of month and Day of week fields are both other than *, the command is executed when either of these two fields are satisfied

| * | always |
| --- | --- |
| A number | A number to specify a number of minutes or hours, a date, or a weekday. For weekdays, 0 equals Sunday, 1 equals Monday, 2 equals Tuesday, and so on. 7 also equals Sunday. |
| x-y |  range, x to y inclusive |
| x,y  | Lists can include ranges as well, for example, 5,10-13,1 |
| */x  | To indicate an interval of x, for example, */7 in the Minutes column runs a job every seven minutes. |

### Anacrontab

The purpose of /etc/anacrontab is to make sure that important jobs always run, and not skipped accidentally because the system was turned off or hibernating when the job should have been executed

Three factors it examines are:

1. The presence of the /var/spool/anacron/cron.daily
2. The elapsed time of 24 hours since it was last run
3. If the system is plugged in to an AC source

If all three conditions are true, Anacron automatically executes the scripts located in the /etc/cron.daily, /etc/cron.weekly, and /etc/cron.monthly directories based on the settings and conditions defined in Anacron’s own configuration file /`etc/anacrontab`

## Exercises

1. When will `/usr/bin/touch test_passed` command run?

    ```bash
    0 3 15 * * /usr/bin/touch test_passed
    ```

    On the 15th of each month, at 3 AM

2. We're logged in as the user called `alex`   How do we see the crontab for the`root` user?
    sudo crontab -l

3. How can we force `anacron` to rerun all jobs, regardless of when they were last executed?
    We can force `anacron` to rerun all jobs, regardless of when they were last executed using `sudo anacron -n -f`

4. What is the command to see the jobs that are scheduled to run in `at utility ?`

   `atq`

5. Remove all `at` jobs that exist for the user `bob`.

    `atrm [ID]`

6. Add this command to the crontab of root /usr/bin/touch test_passed Make it run every day at `21:30`

    ```bash
    30 21 * * * /usr/bin/touch test_passed
    ```

7. Using `root` user, schedule below given command to run at `15:30 August 20 2024` by using `at`

    ```bash
    sudo -i
    at '15:30 August 20 2024'
    /usr/bin/touch atscheduler
    CTRL+D
    ```

8. Add a cron for `bob` user to execute `sudo systemctl restart nginx`  command on `Sundays` every week at `6am`  and `11pm`

    ```bash
    0 6,23 * * 0 sudo systemctl restart nginx
    ```
