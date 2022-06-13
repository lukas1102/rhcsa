chap 1 installation of rhel: 
Custom Installation:
    -> manual partition
        -> LVM not for booting use standard partition for device type

chap 2 linux commands:
    pwd
    whoami
    ls
    ls -l
    ip a s 
    free 
    free -m (MiB)
    df -h 
    cat /etc/hosts
    findmnt
    history
    !11
    !f (last command started with f)
    less
    ps aux
    wc (lines, words, characters)
    ls > lsfiles
    ls >> lsfiles
    ls lsdsfs 2> errors
    ls lwers * 2> /dev/null 
    env | less
    alias

STDIN  -> CMD -> STDOUT
file <        > filename
                STDERR
              2> errors
    
    ls /etc > etcfiles
    less etcfiles
    who 
    who > etcfiles 
    grep -R student /etc 2> /dev/null 

    Standard directories are definded in the FHS: /boot, /home, /var
            mount
    /  <-- /dev/sda2
    /boot <-- /dev/sda1
    /home  <-- server:/home
    /var  <-- /dev/sdb

     cd .. 
    cd /boot
    ll vmlinuz*.el8  (linux kernel)
    cd /dev
    ll
    (major, minor identifier of the kernel)
    cd /etc
    cat passwd
    cat redhat-release

    cat /etc/os-release
    cd /home
    useradd linda
    cd /usr
    /usr/bin (user binaries)
    /usr/sbin (system binaries) 
    /var/log
    man hier
    

    



    
    