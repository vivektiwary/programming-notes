## Command Line Shortcuts:
1. Switching places with the char on cursor
`ctrl + t`

2. Moving forward
`ctrl + f`

3. Moving backward
`ctrl + b`

4. Deleting one char behind
`ctrl + h`

5. Deleting one char in-front 
`ctrl + d`

6. Brings up the last command
`ctrl + p`


## Command line tools:
1. `tail`
Gives you last 10 lines of a file. We can change the number of lines using -n switch

2. `head`
Gives you first 10 lines of a file. We can change the number of lines using -n switch


## Setting variables in bash:
> `export VAR_NAME = VAR_VAL`  
For refering the variable use `$` before variable name.

## Environment variables in Bash:
1. `set`: Dump all the set environment variables
2. `unset`: Unsets the env variable [use env variable with `$` sign
3. For setting any environment variable which are already set, we can use `export ENV_VAR_NAME = ENV_VAR_VAL`

## Customizing the PS(Prompt string) environment variables


## Bash history



## Finding Files in linux
1. `which`: Displays the full path of shell commands
2. `whereis`: Displays the binary, the source and the documentation command

3. `locate`: Shows all the files matching the search. ex: `locate pwd`.  
**Note**: Instead of searching the whole system, `locate` search the __database__.
A cron runs every few minutes and update the `database`.That's why it is
lot faster than the normal `find`.
4. `updatedb`: Updates the database used by locate command

5. `find`: Very powerful tool for searching files using multiple parameters.
    1. `find . -name 'cron*'`: It will search the current directory and sub directory for anything that starts with cron
    2. `find . -type f -name 'cron*'`: Search for `only files` no `directory`.
    3. `find . -type d -name 'cron*'`: Search for `only directories` no `files`.
    4. `find /home -perm 777`: Search in the home directory for the files having permission `777`.
    5. `find /home -perm 777 -exec chmod 555 {} \;`: Change the permission of files having 777 permission to `555`
    6. `find / -mtime +1`: List everything in the file system that has been modified last day. __+1__ for 1 day
    7. `find / -a +1`: List everything which has been accessed in last 1 day
    8. `find / -group nameofgroup`: Find everything in the group
    9. `find . -size 512c`: Find with size
    10. `find . -size 1M`: Search for files having size = 1mb

## Using streams, grep and cut:
1. `wc`: Gives the word count for a file.
    1. For counting the number of lines use `wc -l`
2. `split`: Use to split file based on some params
    1. For split file based on line num, do `split -l no_of_lines FILE_NAME`
3. `diff`: Finding the difference between two files.

---
1. Redirecting standard error to somewhere else: `ls somehtingerror 2> somefile`.
The `2` here says that redirect standard error to that file.
2. Redirecting standard output and standard error to 2 separate files.
`ls someoutput someerror > outputfile 2> errorfile
`
3. Redirecting standard output and standard error to 1 file
`ls someoutput someerror > outputfile 2>&1`
4. `noclobber`: When we set `set -o noclobber`, we can not append or replace content
from already existing files.
    1. To remove it just do `set +o noclobber`
    
    