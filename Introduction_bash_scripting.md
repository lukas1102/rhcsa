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

# Using Loops in Shell Scripts 
- Different conditional statements are available in bash
- `if ... then ... fi`
- `while ... do ... done`
- `until ... do ... done`
- `case ... in ... esac`
- `for ... in ... do ... done`

```
cat > myargument << EOF
#!/bin/bash

if [ -z $1 ]
then 
    echo you have to provide an agrument
    exit 6
fi

echo the argument is $1
EOF
chmod u+x argument 
man test
./myargument
echo $?
./myargument abc
echo $?

cat > countdown << EOF
#!/bin/bash
COUNTER=$1
COUNTER=$(( COUNTER * 60 ))

minusone() {
    COUNTER=$(( COUNTER - 1 ))
    sleep 1
}

while [ $COUNTER -gt 0 ]
do
    echo you have $COUNTER seconds left
    minusone
done
[ $COUNTER = 0 ] && echo time is up && minusone
[ $COUNTER = "-1" ] && echo you are one second late && minusone

while true
do 
    echo you are now ${COUNTER#-} seconds late
    minusone
done
EOF

echo $(( 2 + 4 ))
echo $(( 2 * 4 ))
echo $(( 5 / 2 ))

chmod +x countdown
./countdown 1
```

