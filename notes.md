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
	man man
	man is section based

	sections:
	 1 executes (without root privileges)
	 5 config files
	 8 system admin commands (root privileges)
	 
	 examples near at the bottom (up G) /examples
	synopsis:
	 command structure [option] optional 
						... previous item can be repeated
					  short or long notations
					  SEE ALSO (related commands)
					  
					  -a| --all y|n  

					lvcreate -L|--size Size[m|UNIT] VG
	man lvcreate
	man ls
	mandb
	man -k apropos to search the mandb based on a keyword
	
	man -k user | grep 8


vim:
	start vim -> command mode  - by pressing a,i,o or Ins -> insert mode  - esc -> command mode (:wq!)
	
	i,a,r insert, append, replace
	o open new line
	:wq write and quit
	:q! quit without saving
	dd delete a line
	yy copy current line
	p paste
	d$ removes from cursor positon until end of line
	v visual mode for selection
	u undo
	ctrl +r redo
	gg go to first line
	G go to last line
	/text search top down for text (n for next occurence)
	?test search bottom up
	^ set cursor to start of line
	$ set cursor to end of line
	:%s/old/new/g global substitute of old to new

globbing and wildcards:
globbing is shell feature that helps matching filenames, not confusing with regex
	man 7 glob
	ls /etc/host*
	touch hosts ghosts gosts
	ls ?ost
	ls [hm]ost
	ls [!hm]ost everything that starts not with h,m
	touch script{0..100}
	ls script[0-9][0-9]
	ls /etc/*[0-9]*  -> in globbing if a directory matches a directory, ls will show the content of the directory
	ls -d /etc/*[0-9]* -> dont show directory content
	
	
cockpit (monitoring tool):
	systemctl enable --now cockpit.socket
	http://localhost:9090
	
File managment:	
	ls
	mkdir
	mkdir scripts
	mkdir -p new/scripts 
	cp /etc/hosts .
	cp -r /etc/h* . (+ directories and content)
	mv script* scripts/
	rmdir (only empty directories) 
	rm -rf scripts/ 
finding files:
	which (looks for binaries in a $PATH)
	locate (uses database built by updatedb)
	find 
	
	which useradd
	echo $PATH
	updatedb
	locate useradd
	find / -name "hosts"
	find / -name "host?"
	find / -type f -size +100M  (find files >100M)
	find / -user admin (what files do the user admin own)
	find /etc -exec grep -l student {} \; 2> /dev/null searches in a files the word student
	grep -l student /etc/*
	find /etc -size +100c -exec grep -l student {} \; 2> /dev/null 
	find /etc -size +100c -exec grep -l student {} \; -exec cp {} /tmp \; 2> /dev/null 
	
	
mounting: 
	to access a device, it needs a directory
	linux uses multiple mounts
	different types are on different devices: security, manageability, specific mount options
	
links:
	links are pointer to files in a different location
	ln hard link 
	ln -s symbolic link
	
	name, name2, name3 -> inode(inode counter) -> blocks  (hard link)
	slink1 -> name2 (symbolic link)
	
	symbolic links: cross devices, directories
	ls -il (inode number)
	ln /etc/hosts /root/hardhosts
	ls -il /etc/hosts /root/hardhosts
	ln -s /etc/hosts symhosts  (absolut filename)
	ln -s /var .
	ls -il /etc/hosts /root/hardhosts /root/symhosts
	ls -il
	rm -f /etc/hosts
	ln /root/hardhosts /etc/hosts 
    

tar:
	tape archiver, default does not compress data
	tar -cvf my_archive.tar /home /etc 
	tar -tvf (will show content of archive)
	tar -xvf (extracts to the current directory, tar always stores relative files)
		-C to switch to output path
	for compression use -z, -j or -J
	mv my_archive.tar mytar
	file mytar
	
	compression solutions:
	gzip 
		gzip mytar
		gzip -k mytar  (keeps origin)
		gunzip mytar.gz
	bzip2
		bzip2 -k mytar	
	zip
	xz
		xz -k mytar 

ctrl +a goes to pos1
mkdir /tmp/archive; tar -xvf mytar.xz -C /tmp/archive

common text tools:
	more was original file paper
	less was developed to enhance more
	to do more, use less	

	less /etc/passwd
	less:
		space bar for going down
		arrow keys or page up/down for moving
		q for quit
	more can only go down, not up. it shows the percentage of the page
	head (first 10 lines)
	tail (last 10 lines)
	 -n (number of lines)
	head -n 10 /etc/passwd | tail -n 1 (get only line 10)
	tail -f /var/log/messages
	cat (dumps text file content)
		-A shows all nonprintable characters
		-b numbers lines
		-s suppresses repeated empty lines
	tac (is doing cat in reverse order)
	cat -A /etc/hosts
		(^I tab, $ end of line)
	cut (filters output)
	cut -f 3 -d : /etc/passwd | less (shows 3.line)
	sort (sorts output)
	cut -f 3 -d : /etc/passwd | sort | less 
	cut -f 3 -d : /etc/passwd | sort -n | less (numeric order) 
	tr (translates)
	cut -f 1 -d : /etc/passwd | sort | tr [a-z] [A-Z]
	cut -f 1 -d : /etc/passwd | sort | tr [:lower:] [:upper:] (also works with special chars)

	grep (generic regular expressions parser, for find test in files)
	ps aux | grep ssh
	grep linda * 2> /dev/null
	grep -l linda * 2> /dev/null (show only the files)
	grep -i linda * (case sensitive)
	grep -A4 -B5 linda /etc/passwd (4 lines after the match, 5 lines before match)
	grep -Rl root /etc 2> /dev/null | less (recursive search)

regular expressions: (are text patterns, dont confuse it globbing)
	use regex for text inside a file, and globbing for file names
	grep 'a*' a* (first 'a*' is regex, a* is globbing with out ' the bash is interpreting the regex)
	common tools for regex are grep, vim, awk, sed
	man 7 regex
	regex are built around atoms:
		atoms can be a single char, range of chars or a dot
		atoms can also be a class [[:alpha:]], [[:upper:]], [[:digit:]], [[:alnum:]]
	second is a repition operator, for how often a char occurs
	third element is indication where to find next char
	grep 'b.t' regtext (. one single char) 
	grep 'b.*t' regtext (* repition operator 0 or more times)
	grep 'bo*t' regtext 
	egrep 'b.?t' regtext (extended regular expression ? 0,1 time occurence)
	^ beginning of line
	$ end of line
	\< beginning of word
	\> end of word
	* zero or more times
	+ one or more times
	? zero or one times
	{n} exactly n times
awk: (powerful text processing tool)
	awk -F : '/linda/ { print $4}' /etc/passwd
	awk -F : '{ print $NF}' /etc/passwd (NF for number of fields, printing last field)
	ls -l /etc | awk '/pass/ { print }' | less <--> ls -l /etc | grep pass
