## chap 1 installation of rhel: 
Custom Installation: <br />
    -> manual partition <br />
        -> LVM not for booting use standard partition for device type <br />

## chap 2 linux commands:
pwd <br />
whoami <br />
ls <br />
ls -l <br />
ip a s  <br />
free  <br />
free -m (MiB) <br />
df -h  <br />
cat /etc/hosts <br />
findmnt <br />
history <br />
!11 <br />
!f (last command started with f) <br />
less <br />
ps aux <br />
wc (lines, words, characters) <br />
ls > lsfiles <br />
ls >> lsfiles <br />
ls lsdsfs 2> errors <br />
ls lwers * 2> /dev/null  <br />
env | less <br />
alias <br />
 <br />
STDIN  -> CMD -> STDOUT <br />
file <        > filename <br />
                STDERR <br />
              2> errors <br />
     <br />
ls /etc > etcfiles <br />
less etcfiles <br />
who  <br />
who > etcfiles  <br />
grep -R student /etc 2> /dev/null  <br />
 <br />
*Standard directories are definded in the FHS(Filesystem Hierarchy Standard): /boot, /home, /var* <br />
        mount <br />
/  <-- /dev/sda2 <br />
/boot <-- /dev/sda1 <br />
/home  <-- server:/home <br />
/var  <-- /dev/sdb <br />
 <br />
cd ..  <br />
cd /boot <br />
ll vmlinuz*.el8  (linux kernel) <br />
cd /dev <br />
ll <br />
(major, minor identifier of the kernel) <br />
cd /etc <br />
cat passwd <br />
cat redhat-release <br />
 <br />
cat /etc/os-release <br />
cd /home <br />
useradd linda <br />
cd /usr <br />
/usr/bin (user binaries) <br />
/usr/sbin (system binaries)  <br />
/var/log <br />
### man
man hier <br />
man man <br />
man is section based <br />
 <br />
sections: <br />
 1 executes (without root privileges) <br />
 5 config files <br />
 8 system admin commands (root privileges) <br />
	  <br />
examples near at the bottom (up G) /examples <br />
synopsis: <br />
command structure [option] optional  <br />
					... previous item can be repeated <br />
				  short or long notations <br />
				  SEE ALSO (related commands) <br />
					   <br />
					  -a| --all y|n   <br />
 <br />
					lvcreate -L|--size Size[m|UNIT] VG <br />
man lvcreate <br />
man ls <br />
mandb <br />
man -k apropos to search the mandb based on a keyword <br />
	 <br />
man -k user | grep 8 <br />

### vim:
start vim -> command mode  - by pressing a,i,o or Ins -> insert mode  - esc -> command mode (:wq!) <br />
	 <br />
i,a,r insert, append, replace <br />
o open new line <br />
:wq write and quit <br />
:q! quit without saving <br />
dd delete a line <br />
yy copy current line <br />
p paste <br />
d$ removes from cursor positon until end of line <br />
v visual mode for selection <br />
u undo <br />
ctrl +r redo <br />
gg go to first line <br />
G go to last line <br />
/text search top down for text (n for next occurence) <br />
?test search bottom up <br />
^ set cursor to start of line <br />
$ set cursor to end of line <br />
:%s/old/new/g global substitute of old to new <br />

### globbing and wildcards:
globbing is shell feature that helps matching filenames, not confusing with regex <br />
man 7 glob <br />
ls /etc/host* <br />
touch hosts ghosts gosts <br />
ls ?ost <br />
ls [hm]ost <br />
ls [!hm]ost everything that starts not with h,m <br />
touch script{0..100} <br />
ls script[0-9][0-9] <br />
ls /etc/*[0-9]*  -> in globbing if a directory matches a directory, ls will show the content of the directory <br />
ls -d /etc/*[0-9]* -> dont show directory content <br />
	
	
### cockpit (monitoring tool):
systemctl enable --now cockpit.socket <br />
http://localhost:9090
	
### File managment:	
ls <br />
mkdir <br />
mkdir scripts <br />
mkdir -p new/scripts  <br />
cp /etc/hosts . <br />
cp -r /etc/h* . (+ directories and content) <br />
mv script* scripts/ <br />
rmdir (only empty directories)  <br />
rm -rf scripts/  <br />
### finding files:
which (looks for binaries in a $PATH) <br />
locate (uses database built by updatedb) <br />
find  <br />
	 <br />
which useradd <br />
echo $PATH <br />
updatedb <br />
locate useradd <br />
find / -name "hosts" <br />
find / -name "host?" <br />
find / -type f -size +100M  (find files >100M) <br />
find / -user admin (what files do the user admin own) <br />
find /etc -exec grep -l student {} \; 2> /dev/null searches in a files the word student <br />
grep -l student /etc/* <br />
find /etc -size +100c -exec grep -l student {} \; 2> /dev/null  <br />
find /etc -size +100c -exec grep -l student {} \; -exec cp {} /tmp \; 2> /dev/null  <br />
	
### mounting: 
to access a device, it needs a directory <br />
linux uses multiple mounts <br />
different types are on different devices: security, manageability, specific mount options <br />
	
### links:
links are pointer to files in a different location <br />
ln hard link  <br />
ln -s symbolic link <br />
	 <br />
name, name2, name3 -> inode(inode counter) -> blocks  (hard link) <br />
slink1 -> name2 (symbolic link) <br />
	 <br />
*symbolic links: cross devices, directories* <br />
ls -il (inode number) <br />
ln /etc/hosts /root/hardhosts <br />
ls -il /etc/hosts /root/hardhosts <br />
ln -s /etc/hosts symhosts  (absolut filename) <br />
ln -s /var . <br />
ls -il /etc/hosts /root/hardhosts /root/symhosts <br />
ls -il <br />
rm -f /etc/hosts <br />
ln /root/hardhosts /etc/hosts  <br />

### tar:
tape archiver, default does not compress data <br />
tar -cvf my_archive.tar /home /etc  <br />
tar -tvf (will show content of archive) <br />
tar -xvf (extracts to the current directory, tar always stores relative files) <br />
	-C to switch to output path <br />
for compression use -z, -j or -J <br />
mv my_archive.tar mytar <br />
file mytar <br />
	 <br />
compression solutions: <br />
gzip  <br />
	gzip mytar <br />
	gzip -k mytar  (keeps origin) <br />
	gunzip mytar.gz <br />
bzip2 <br />
	bzip2 -k mytar	 <br />
zip <br />
xz <br />
	xz -k mytar  <br />

*ctrl +a goes to pos1*
mkdir /tmp/archive; tar -xvf mytar.xz -C /tmp/archive

### common text tools:
more was original file paper <br />
less was developed to enhance more <br />
to do more, use less	 <br />
 <br />
less /etc/passwd <br />
less: <br />
	space bar for going down <br />
	arrow keys or page up/down for moving <br />
	q for quit <br />
more can only go down, not up. it shows the percentage of the page <br />
head (first 10 lines) <br />
tail (last 10 lines) <br />
 -n (number of lines) <br />
head -n 10 /etc/passwd | tail -n 1 (get only line 10) <br />
tail -f /var/log/messages <br />
cat (dumps text file content) <br />
	-A shows all nonprintable characters <br />
	-b numbers lines <br />
	-s suppresses repeated empty lines <br />
tac (is doing cat in reverse order) <br />
cat -A /etc/hosts <br />
	(^I tab, $ end of line) <br />
cut (filters output) <br />
cut -f 3 -d : /etc/passwd | less (shows 3.line) <br />
sort (sorts output) <br />
cut -f 3 -d : /etc/passwd | sort | less  <br />
cut -f 3 -d : /etc/passwd | sort -n | less (numeric order)  <br />
tr (translates) <br />
cut -f 1 -d : /etc/passwd | sort | tr [a-z] [A-Z] <br />
cut -f 1 -d : /etc/passwd | sort | tr [:lower:] [:upper:] (also works with special chars) <br />

### grep (generic regular expressions parser, for find test in files)
ps aux | grep ssh <br />
grep linda * 2> /dev/null <br />
grep -l linda * 2> /dev/null (show only the files) <br />
grep -i linda * (case sensitive) <br />
grep -A4 -B5 linda /etc/passwd (4 lines after the match, 5 lines before match) <br />
grep -Rl root /etc 2> /dev/null | less (recursive search) <br />

## regular expressions: 
*are text patterns, dont confuse it globbing*
use regex for text inside a file, and globbing for file names <br />
grep 'a*' a* (first 'a*' is regex, a* is globbing with out ' the bash is interpreting the regex) <br />
common tools for regex are grep, vim, awk, sed <br />
man 7 regex <br />
regex are built around atoms: <br />
	- atoms can be a single char, range of chars or a dot <br />
	- atoms can also be a class [[:alpha:]], [[:upper:]], [[:digit:]], [[:alnum:]] <br />
second is a repition operator, for how often a char occurs <br />
third element is indication where to find next char <br />
grep 'b.t' regtext (. one single char)  <br />
grep 'b.*t' regtext (* repition operator 0 or more times) <br />
grep 'bo*t' regtext  <br />
egrep 'b.?t' regtext (extended regular expression ? 0,1 time occurence) <br />
^ beginning of line <br />
$ end of line <br />
\< beginning of word <br />
\> end of word <br />
* zero or more times <br />
+ one or more times <br />
? zero or one times <br />
{n} exactly n times <br />
### awk: (powerful text processing tool) <br />
awk -F : '/linda/ { print $4}' /etc/passwd <br />
awk -F : '{ print $NF}' /etc/passwd (NF for number of fields, printing last field) <br />
ls -l /etc | awk '/pass/ { print }' | less <--> ls -l /etc | grep pass <br />
