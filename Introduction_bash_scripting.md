# Understanding Bash Shell scripts
- A shell script can be as simple as a number of commands that is squentially executed
- Scripts normally work with variables to make them react differently in different environments
- Conditional statements such as for, if, case, and while can be used
- shell scripts are common as they are easy to learn and implement
- Also, a shell will always be a available to interpret code from shell scripts
- if the scripts use internal commands only, they are very fast as nothing needs to be loaded
- There is no need to compile anything
- There are no modules to be used in the bash script, which makes them rather static
- Also, Bash shell scripts are not idempotent

# Essential shell script components
```
ls -l myscript
./myscript
chmod +x myscript
./myscript
myscript
echo $PATH
cat > myscript << EOF
#!/bin/bash
# every shell script should have some comment
echo which directory do you want to activate?
read DIR
cd $DIR
pwd
ls
EOF
./myscript
pwd
. myscript
source myscript
pwd

```