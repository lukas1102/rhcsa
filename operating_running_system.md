# Managing processes
## understanding jobs, processes
- all tasks are started as processes
- processe hava a PID
- common process management tasks include scheduling priority and sending signals
- some processes are starting multiple threads, individual threads cannot be managed
- tasks that are managed from a shell can be managed as jobs
- jobs can started in the foreground or background

### managing shell jobs
- use `command &` to start a job in the background
to move a job to the background 
-- first stop it using `ctrl+z`
-- next, type `bg` to move it to the background
- use `jobs` for a complete overview of running jobs
- use `fg [n]` to move the last job back  to the foreground
```
while true; do true; done &
dd if=/dev/zero of=/dev/null
^Z
bg
jobs
fg 1
^C
```
### getting process information with ps
- `ps` stands for process
- the `ps` command has two different dialects: BSD and System5
- Therefore `ps -L` and `ps L` are two completely different commands!
- `ps` shows an overview of current processes
- use `ps aux` for an overview of all processes
- `ps -fax` shows hierachical relations between processes
- `ps -fU linda` shows all processes owned by linda
- `ps -f --forest -C sshd` shows a process tree for a specific process
- `ps L` shows format specifier
- `ps -eo pid,ppid,user,cmd` uses some of these specifiers to show a list of processes
```
ps L
ps -L
ps
ps aux | less  (processes listed in brackets are kernel processes)
ps -fax | less
ps -fU student | less
ps -f --forest -C sshd
ps L | wc
ps -eo pid,ppid,user,cmd
```
### understanding memory usage
- Linux places as many files as possible in cache to guarantee fast access to the files
- For that reason, Linux memory often shows as saturated
- Swap is used as an overflow buffer of emulated RAM on disk
- The Linux kernel moves inactive memory to swap first
- use `free -m` to get details about current memory usage
### understanding CPU load

some task  -->  runqueue  
                pid 238
                pid 237
                pid 236

    CPU 0 <-- scheduler  --> CPU 1

`uptime` load average: 0.00,0.00,0.00 (average load of last minute, last 5min, last 15min)
```
dd if=/dev/zero of=/dev/null &
watch uptime
lscpu
```

### Monitoring system activity with top
- `top` is a dashboard that allows you to monitor current system activity
- press `f` to show and select from available display fields
- type `M` to filter on memory usage
- Press `W` to save new display settings
- Press `k` to send signals to a process
- Press `r` to renice a process

### sending signals to processes
- A signal allows the operating system to interrupt a process from software and ask it to do something
- interrupts are comparable to signals, but are generated from hardware
- a limited amount of signals can be used and is documented in `man 7 signals`
- not all signals work in all cases
- the `kill` command is used to send signals to PIDs
- you can use `k` from top
- Different kill-like commands exist, like `pkill` and `killall`

```
man 7 signals
kill --help | less
ps aux | grep dd
kill -9 14273
top
pkill --help
killall --help
ps aux | grep '\<dd'
killall dd
```

### Managing Priorities and Niceness
- By default Linux processes are started with the same priority
- In kernel-land real-time processes can be started, which will always be handled with highest priority
- to change priorities of non realtime processes, the nice and renice commands can be used
- nice values range from -20 up to 19
- negative nice value indicates an creased priority, a positive nice value inidcates decreased priority
- users can set their process to a lower priority, to increase priorities you need root access

```
dd if=/dev/zero of=/dev/null &
dd if=/dev/zero of=/dev/null &
dd if=/dev/zero of=/dev/null &
top
nice --help
nice -n -1 dd if=/dev/zero of=/dev/null &
renice --help
renice -n 10 14740
```

### using tuned profiles
- `tuned` is a service tht allows for performance optimization in an easy way
- different profiles are provided to match specific server workloads
- to use them, ensure the `tuned` service is enabled and started
- `tuned-adm list` will show a list of profiles
- `tuned-adm profile <name>` will set a profile
- `tuned-adm active` will show the current profile

```
systemctl status tuned
tuned-adm list
tuned-adm profile desktop
tuned-adm active
```