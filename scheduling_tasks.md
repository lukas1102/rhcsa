# understanding Cron and at
- Cron is a daemon that triggers jobs on a regular basis
- it works with different configuration files that specify when a job should be started
- use it for regular re-occurring jobs, like backup jobs
- `at` is used for tasks that need to be started once. It can be started via command line, no configuration is needed
- systemd timers provide a new alternative to Cron

# understanding Cron scheduling options
- user-specific Cron jobs, created using `crontab -e`
- generic time-specific Cron jobs in /etc/cron.d
- Scripts, executed on an hourly, daily, weekly, monthly basis
- Generic time-specific Cron jobs in /etc/crontab (deprecated)

# understanding anacron
- Anacron is a service behind cron that takes care jobs are executed on a regular basis, but not at a specific time
- It takes care of the jobs in /etc/cron.{hourly,daily,weekly,monthly}
- Configuration is in /etc/anacrontab

# Scheduling with cron
- Choose the option you want to use
- - `crontab -e` as a specific user
- - Create a cron file in /etc/cron.d
- Use the time specification: */10 4 11 12 1-5
- - minute (*/10)
- - hour (4)
- - day of month (11)
- - month (12)
- - day of week (1-5)
- Note that cron does NOT have STDOUT

`man 5 crontab`

# Scheduling tasks with Systemd timers
- systemd timers also allow for scheduling jobs at a regular basis, Cron however is still the standard
- read `man 7 systemd-timer` for more information about systemd timers
- read `man 7 systemd-time` for specification of the time format to be used
```
cd /usr/lib/systemd/system
ls *timer
ls fstrim*
cat fstrim.timer
cat fstrim.service
systemctl status fstrim.service
systemctl status fstrim.timer
systemctl enable fstrim.timer
systemctl start fstrim.timer
```

# Using at
- The `atd` service must be running to run once-only jobs using `at`
- Use `at <time>` to schedule a job
- - type one or more job specifications in the at interactive shell
- - Use Ctrl-D to close this shell
- Use atq for a list of jobs currently scheduled
- Use atrm to remove jobs from the list

```
systemctl status atd
at teatime
> logger have a cup of tea
> mail -s hello root < .
> <EOT>

atq
atrm 1
atq
```

# Managing temporary files
## understanding systemd-tmpfiles
- The /usr/lib/tmpfiles.d directory manages settings for creating, deleting and cleaning up of temporary files
- The `systemd-tmpfiles-clean.timer` unit can be configured to automatically clean up temporary files
- - it triggers the `systemd-tmpfiles-clean.service`
- - this service runs `systemd-tmpfiles --clean`
- The /usr/lib/tmpfiles.d/tmp.conf file contains settings for the automatic tmp file cleanup
- When making modifications, copy the file to /etc/tmpfiles.d
- After making modifications to this file, use `systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf` to ensure the file does not contain any errors

## understanding tmp.conf
- in tmp.conf you'll find lines specifiying which directory to monitor, which permissions are set on that directory, which owners, and after how many days of not being used the tmp files will be removed
- different actions can be performed on the directories and files that are managed
- consult `man tmpfiles.d` for more details

```
systemctl cat systemd-tmpfiles-clean.timer
cp /usr/lib/tmpfiles.d/tmp.conf /etc/tmpfiles.d/
vim /etc/tmpfiles.d/tmp.conf
man tmpfiles.d
systemd-tmpfiles --clean /etc/tmpfiles.d/tmp.conf
echo "d /run/mytemp 0700 root root 30s" > /etc/tmpfiles.d/mytemp.conf
ls /etc/tmpfiles.d/
ls /usr/lib/tmpfiles.d/
systemd-tmpfiles --create /etc/tmpfiles.d/mytemp.conf
touch /run/mytemp/myfile
ls /run/mytemp/myfile
sleep 30
ls /run/mytemp/myfile
systemd-tmpfiles --clean /etc/tmpfiles.d/mytemp.conf
ls /run/mytemp/myfile
```