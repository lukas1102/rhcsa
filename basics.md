## Installation of RHEL: 
Custom Installation: <br />
&nbsp;&nbsp;&nbsp;&nbsp;-> manual partition <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-> LVM not for booting use standard partition for device type <br />

## Linux commands:
```
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
```

STDIN  -> CMD -> STDOUT <br />
file <        > filename <br />
&nbsp;&nbsp;&nbsp;&nbsp;STDERR <br />
&nbsp;&nbsp;&nbsp;&nbsp;2> errors <br />

```
ls /etc > etcfiles 
less etcfiles 
who  
who > etcfiles  
grep -R student /etc 2> /dev/null  
```

*Standard directories are definded in the FHS(Filesystem Hierarchy Standard): /boot, /home, /var* <br />
&nbsp;&nbsp;&nbsp;&nbsp;mount <br />
/  <-- /dev/sda2 <br />
/boot <-- /dev/sda1 <br />
/home  <-- server:/home <br />
/var  <-- /dev/sdb <br />

 ```
cd ..  
cd /boot 
ll vmlinuz*.el8  (linux kernel)
cd /dev 
ll 
```

*(major, minor identifier of the kernel)* <br />

```
cd /etc 
cat passwd 
cat redhat-release 

cat /etc/os-release 
cd /home 
useradd linda 
cd /usr 
/usr/bin *(user binaries)* 
/usr/sbin *(system binaries)*  
/var/log 
```

### man
```
man hier 
man man 
```
man is section based  <br />

sections: <br />
 1 executes (without root privileges) <br />
 5 config files <br />
 8 system admin commands (root privileges) <br />
	 
examples near at the bottom (up G) /examples <br />
synopsis: <br />
command structure [option] optional  <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;... previous item can be repeated <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;short or long notations <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SEE ALSO (related commands) <br />	
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-a| --all y|n   <br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lvcreate -L|--size Size[m|UNIT] VG <br />

```
man lvcreate 
man ls 
mandb 
man -k apropos (to search the mandb based on a keyword)

man -k user | grep 8 
```

### vim:
start vim -> command mode  - by pressing a,i,o or Ins -> insert mode  - esc -> command mode (:wq!) <br />

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
*globbing is a shell feature that helps matching filenames, not confusing with regex* <br />

```
man 7 glob 
ls /etc/host* 
touch hosts ghosts gosts 
ls ?ost 
ls [hm]ost 
ls [!hm]ost (everything that starts not with h,m) 
touch script{0..100} 
ls script[0-9][0-9] 
ls /etc/*[0-9]*  -> in globbing if a directory matches a directory, ls will show the content of the directory 
ls -d /etc/*[0-9]* -> dont show directory content 
```
	
### cockpit (monitoring tool):
`systemctl enable --now cockpit.socket` <br />
http://localhost:9090
	
### File managment:	
```
ls 
mkdir 
mkdir scripts 
mkdir -p new/scripts  
cp /etc/hosts . 
cp -r /etc/h* . (+ directories and content) 
mv script* scripts/ 
rmdir (only empty directories)  
rm -rf scripts/  
```
### finding files:
```
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
```
	
### mounting: 
to access a device, it needs a directory <br />
linux uses multiple mounts <br />
different types are on different devices: security, manageability, specific mount options <br />
	
### links:
*links are pointer to files in a different location* <br />

ln hard link  <br />
ln -s symbolic link <br />

name, name2, name3 -> inode(inode counter) -> blocks  (hard link) <br />
slink1 -> name2 (symbolic link) <br />

*symbolic links work across devices and directories* <br />
```
ls -il (inode number) 
ln /etc/hosts /root/hardhosts 
ls -il /etc/hosts /root/hardhosts 
ln -s /etc/hosts symhosts  (absolut filename) 
ln -s /var . 
ls -il /etc/hosts /root/hardhosts /root/symhosts 
ls -il 
rm -f /etc/hosts 
ln /root/hardhosts /etc/hosts  
```

### tar:
*tar archiver, default does not compress data* <br />
```
tar -cvf my_archive.tar /home /etc  
tar -tvf (will show content of archive) 
tar -xvf (extracts to the current directory, tar always stores relative files) 
&nbsp;&nbsp;&nbsp;&nbsp;-C to switch to output path 
for compression use -z, -j or -J 
mv my_archive.tar mytar 
file mytar 
```

compression solutions: <br />
gzip  <br />
&nbsp;&nbsp;&nbsp;&nbsp;gzip mytar <br />
&nbsp;&nbsp;&nbsp;&nbsp;gzip -k mytar  (keeps origin) <br />
&nbsp;&nbsp;&nbsp;&nbsp;gunzip mytar.gz <br />
bzip2 <br />
&nbsp;&nbsp;&nbsp;&nbsp;bzip2 -k mytar	 <br />
zip <br />
xz <br />
&nbsp;&nbsp;&nbsp;&nbsp;xz -k mytar  <br />

*ctrl +a goes to pos1* <br />
`mkdir /tmp/archive; tar -xvf mytar.xz -C /tmp/archive`

### common text tools:

more was original file paper  <br />
less was developed to enhance more  <br />
to do more, use less	 <br />

`less /etc/passwd `  <br />
`less:  ` <br />
space bar for going down    <br />
arrow keys or page up/down for moving   <br />
q for quit   <br />
more can only go down, not up. it shows the percentage of the page   <br />
 ```
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
```

### grep (generic regular expressions parser, for find text in files)
```
ps aux | grep ssh 
grep linda * 2> /dev/null 
grep -l linda * 2> /dev/null (show only the files) 
grep -i linda * (case sensitive) 
grep -A4 -B5 linda /etc/passwd (4 lines after the match, 5 lines before match) 
grep -Rl root /etc 2> /dev/null | less (recursive search) 
```

## regular expressions: 
are text patterns, dont confuse it globbing <br />
use regex for text inside a file, and globbing for file names <br />
`grep 'a*' a*`  (first 'a\*' is regex, a\* is globbing with out ' the bash is interpreting the regex) <br />
common tools for regex are grep, vim, awk, sed <br />
`man 7 regex` <br />
regex are built around atoms: <br />
	- atoms can be a single char, range of chars or a dot <br />
	- atoms can also be a class [[:alpha:]], [[:upper:]], [[:digit:]], [[:alnum:]] <br />
second is a repition operator, for how often a char occurs <br />
third element is indication where to find next char <br />
```
grep 'b.t' regtext (. one single char)  
grep 'b.*t' regtext (* repition operator 0 or more times) 
grep 'bo*t' regtext  
egrep 'b.?t' regtext (extended regular expression ? 0,1 time occurence) 
grep '\<root\>' * 2> /dev/null (searches for the word root)
grep '\<alex\>' * 2> /dev/null 
grep '^alex$' * 2> /dev/null 
egrep '^.{3}$' * 2> /dev/null  	
```
^ beginning of line <br />
$ end of line <br />
\\< beginning of word <br />
\\> end of word <br />
\* zero or more times <br />
\+ one or more times <br />
? zero or one times <br />
\{n\} exactly n times <br />

### awk: (powerful text processing tool) 
```
awk -F : '{ print $1 " \t" $4 }' /etc/passwd 
awk -F : '/linda/ { print $4 }' /etc/passwd 
awk -F : '{ print $NF}' /etc/passwd (NF for number of fields, printing last field) 
ls -l /etc | awk '/pass/ { print }' | less <--> ls -l /etc | grep pass 
```

### sed screen editor
*sed is a stream editor, used to search and transform text*
```
vim sedfile
sed -n 4p sedfile  (prints the 4th line of the file)
sed -i s/four/FOUR/g sedfile  (-i writes directly to the file, not stdout)
sed -i -e '2d' sedfile			(-e passes an edit command to sed; deletes the 2nd line)
```