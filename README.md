# wildwest
- Turn Ansible into a parallel ssh like tool
- Speed optimized configuration with Mitogen
- Automatic logging to readable text file
- Run Ansible tasks and roles directly without Playbook
- Run shell script directly without Playbook

## Usage
Run `ww -h` to get help.

ww uses CCZE internally to colorize logs.  
Read the code here to see what words will be highlighted in different colors: here: https://github.com/cornet/ccze/blob/master/src/ccze-wordcolor.c

## Installation
Tested on Debian 11

Enable global python-argcomplete:  
`sudo activate-global-python-argcomplete3`  

## Basic Scripting Guide
All playbooks and shell-scripts should always have this basic code in the header:  

Playbook:  
`#info: Short description of your script here`  
`#arg: scripts first argument here`  
`#arginfo: Tell what happens when the parameter is passed.`  
`---`  
`rest of the code here...`

Shell Script:  
`#!/bin/sh`  
`#info: Short description of your script here`  
`#arg: scripts first argument here`  
`#arginfo: Tell what happens when the parameter is passed.`  
`rest of the code here...`

After `#arg:` if the passed string only uses characters, ww will parse it as as a fixed parameter.  
If the string is inside angle brackets `<>`, ww will parse it as a variable parameter.

Adding the line: '#autoroot' to the header will make the script always run as root.

Make sure the headers tag lines "#xxx:" are not longer than 80 characters.  
Look at existing scripts to get a better understanding before you write your own.

## Scripting Tips

* Try to follow *Eric Raymond's 17 Unix Rules*.  
https://en.wikipedia.org/wiki/Unix_philosophy  
https://medium.com/programming-philosophy/eric-raymond-s-17-unix-rules-399ac802807

* Preferably write Ansible playbooks in favor of shell-script where you can. Playbooks are more stable and reliable.

* All scripts should be strictly none-interactive.

* Write scripts that are easily readable by human beings. Do not try to be clever or crunch all code into one line just because you can. Be a showoff by impressing your colleges not your computer.

* Write scripts with adhere to the default behavior of stderr and stdout. Meaning don't write scripts that spit out errors where there are none. And the other way around.

* Avoid unnecessary output. Try to use silent confirmation often. Meaning if the requests action/task worked, do not print out unnecessary confirmations. This will keep log-files readable. Especially if you run a lot of tasks. If something went wrong you will know because Ansible will properly parse stderr.

* Where possible try to write portable Posix compliant shell-script. Avoid Bash-ism. This will make sure all scripts can be run on many different systems. The modern shell-script interpreter is usually Dash. To find your default /bin/sh interpreter run: ```readlink -f `which sh` ``` Additionally Dash is many times faster than Bash. You can also try the posh shell for testing.

* Do not write buggy shell-code. Tip: Open .sh scripts in PyCharm and add #!/bin/sh to the first line. In PyCharm install the Shell Script coding assistance plugin.  
Now fix all issues the coding assistance points out. If you use another IDE take a look at ShellCheck, Shfmt and Explainshell.