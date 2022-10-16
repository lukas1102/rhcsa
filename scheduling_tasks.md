# understanding cron and at
- cron is a daemon that triggers jobs on a regular basis
- it works with different configuration files that specify when a job should be started
- use it for regular re-occurring jobs, like backup jobs
- `at` is used for tasks that need to be started once. It can be started via command line, no configuration is needed
- systemd timers provide a new alternative to cron

# understanding cron scheduling options