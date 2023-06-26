# wildwest(1) -- Turn Ansible into a better automation management tool

Wildwest is a wrapper for Ansible, the goal is to make Ansible easier to use.
It also adds several new features to Ansible which makes it easier to use
Ansible as a general purpose automation tool.

- Turn Ansible into a parallel-ssh like tool
- Less verbose output via custom Anstomlog
- Colorized output / logs and save to an HTML file
- TAB Autocomplete playbooks, tasks and scripts
- Extensive help system as part of the CLI
- Speed optimized configuration with Mitogen
- Run Ansible tasks and roles directly without Playbook
- Run shell scripts directly without Playbook, no need to know Ansible

At runtime parameters, do not exist in wildwest. This is a design choice
because automation should be predictable and only be defined by code. Without
runtime parameters there is less room for input errors. It forces the user to
write proper playbooks or scripts instead.

## INSTALLATION

Tested on Debian 11

To enable TAB complete for ww, install python-argcomplete3 and then enable it:

    $ sudo activate-global-python-argcomplete3

## CONFIGURATION

Default configuration can be found in:  
**/usr/local/share/wildwest/ww_default.cfg**  
Open this file to get a better understanding of how wildwest facilitates
namespace and inventory management.

The current user configuration is stored in ~/.ww/ww.cfg

The default Ansible configuration can be found in:  
**/usr/local/share/wildwest/ansible_default.cfg**  
Open this file for more information.

The default settings are optimized for any generic workstation, and connecting
up to around 1000 hosts simultaniously.

## USAGE

### HELP

`ww -h` : This will list the global help for wildwest and also list all
available namespaces and what they do.

`ww <WILDWEST_NAMESPACE> -h`, `ww <WILDWEST_NAMESPACE>` : Help for a specific
namespace. This will list what each playbook, task or script does inside a
specific namespace. Running `ww -t <HOST_OR_GROUP> <WILDWEST_NAMESPACE> -h`
Works as well.

Wildwest will look for a file named 'ww.txt' in the main directory of your
role, collection or folder. Add a very short description of your play to this
file. This will then be displayed in this help interface.

To enable information about each script or task to the help interface, please
add the tag '#info: \<TEXT\>' to your files. All playbooks and shell-scripts
should always have this basic code in the header:

Playbook, task or role:

```
---
#info: Short description of your script here
rest of the code here...
```

Shell script:

```
#!/bin/sh
#info: Short description of your script here
rest of the code here...
```

### SYNTAX

`ww [<OPTION>] [<COMMAND>] [-t <WILDWEST_NAMESPACE> [-b, --become] <ACTION>]`

* `WILDWEST_NAMESPACE` : Extended version of the Ansible namespace system. To
  see how the extended namespaces works, open the file:  
  /usr/local/share/wildwest/ww_default.cfg
* `-b`, `--become` : Force to run the play as superuser. This is useful because
  the BECOME directive is usually not defined in tasks or scripts. Useful when
  developing tasks or scripts.

**Commands:**

If no command is given, wildwest assumes you want to use -h or -t.

* `edin` : Edit the inventory file of Inventorymaker with Visidata.
* `gen` : Force to regenerate the inventory file.
* `exe` : Directly execute a shell command. Will execute with the default sh
  shell on the target system. Only use this command for testing and
  development. It is not a good idea to use ad-hock commands in production.
  When running 'exe' with the '--become' flag on 'localhost', wildwest will
  always ask for password.

**Options:**

* `-t`, `--target <HOST>` : Is the name of the host or host group in the
  Ansible inventory. If you use inventorymaker, the group name can also be the
  same as the inventory CSV file name. You can also use Ansible patterns like '
  host3,host10' or 'host*', see:
  https://docs.ansible.com/ansible/latest/inventory_guide/intro_patterns.html
* `-d`, `--dryrun` : Simulate the Ansible run.
* `-f`, `--format` : Pipe and redirect friendly output. No hidden special
  characters or colors in output.
* `-n`, `--notify` : Play a notification sound when done. Uses aplay.
* `-v`, `--verbose` : Get extra information. Useful if you have to iterate over
  lots of targets. -v is recommended for users, -vv is basic debugging, -vvv
  and -vvvv is for full debugging. This will also show the duration of a play.
* `-w`, `--watch <SECONDS>` : Repeat the same play and watch the output. In
  seconds.
* `-x`, `--export` : Save the script output to a colorful HTML file. Will be
  saved to ~/.ww/reports. Only works with roles 'servermonkey.sh' and '
  servermonkey.ww_logger'.  
  See: [github.com/ServerMonkey/ansible-role-servermonkey-sh](https://github.com/ServerMonkey/ansible-role-servermonkey-sh)  
  and [github.com/ServerMonkey/ansible-role-servermonkey-ww-logger](https://github.com/ServerMonkey/ansible-role-servermonkey-ww-logger)

### CRONTAB

`ww-task <TARGET> <NAMESPACE> <ACTION>` : Wrapper for ww. Starts wildwest
actions in background, use this in crontab. Always Uses become. Will write a
logfile to "~/.ww/cache/logs_task"

## ADDITIONAL FEATURES

wildwest comes with an extra logging system that can be used via this role:  
[github.com/ServerMonkey/ansible-role-servermonkey-ww-logger](https://github.com/ServerMonkey/ansible-role-servermonkey-ww-logger)

When you run a play with servermonkey.ww_logger you will get additional output
to your shell after Ansible has been run. This is useful when you use wildwest
as an administration tool on systems that already have been deployed/installed.

servermonkey.ww_logger is automatically enabled when you run shell scripts
directly. This is also true when running command via the 'exe' command. See:  
[github.com/ServerMonkey/ansible-role-servermonkey-sh](https://github.com/ServerMonkey/ansible-role-servermonkey-sh)

Wildwest uses ccze internally to colorize these logs.  
Read the code here to see what words will be highlighted in different colors:  
[github.com/cornet/ccze/blob/master/src/ccze-wordcolor.c](https://github.com/cornet/ccze/blob/master/src/ccze-wordcolor.c)  
or use my CLI tool ccze-test.

When using the extra logging system. Wildwest will automatically try to make
the output readable by adding extra line-breaks when each host gives more than
one line of output. Practically this means when you run "echo hello" on 10
hosts, you will get 10 lines of output neatly stacked under each other. When
you run a command that gives more than one line of output on each host,
wildwest will automatically add extra line-breaks.

## EXAMPLES

First edit the file ~/.ww/ansible.cfg and add the namespace to enable. As an
example we use my Ansible role from here:  
[github.com/ServerMonkey/ansible-role-servermonkey-ww](https://github.com/ServerMonkey/ansible-role-servermonkey-ww)

Run a playbook on a single hosts in the inventory:

    $ ww -t debianbox servermonkey.ww info_test

Run a playbook on multible hosts and show the duration of the play:

    $ ww -vt debianbox,windowsbox servermonkey.ww info_os

Run a shell command and export the result to an HTML file:

    $ ww -xt debianbox exe "echo Hello World"

Run a shell command as superuser and play a sound when done:

    $ ww -nt debianbox exe -b "sleep 2 ; echo Hello World"

Regernate the inventory because you changed a password:

    $ ww gen

## SEE ALSO

ansible(1), inventorymaker(1), pass(1)
