# Chapter 3 - Implementing Playbooks

## Writing and Runnig Playbooks

### Ansible Playbooks and Ad Hoc Commands
* Ad Hoc commands can run sigle commands against a host but the real power of Ansible is running cmplex tasks using playbooks
* A *Play* is an ordered set of tasks.  A *playbook* is a text file containing one or more plays

### Formatting an Ansible Playbook

* consider the below snipped and how it can be rewritten as a play
```
ansible -m user -a "name=newbie uid=4000 state=present" servera.lab.example.com
```

```
---
- name: Configure important user consistently
  hosts: servera.lab.example.com
  tasks:
    - name: newbie exists with UID 4000
	  user:
	    name: newbie
		uid: 4000
		state: present
```

* A playbook is a text file in YAML format (.yml)
* A Playbook begins with a **---** and may end with **---** as an EOF marker (Optional)
* A play itself is a collection of key-value pairs.  Keys in the same play should have the same indentations
* This example has three keys, *name*, *hosts*, and *tasks*
* The *task* attribute lists in order the tasks to be run on the managed hosts
	* *user* is the module run in this task
* You can have multiple *tasks* in a play

### Running Playbooks
* The **ansible-playbook** command is used to run playbooks
* It is run from the control node and takes the name of the playbook to be run as an argument
'''
ansible-playbook site.yml
'''

#### Increasing Output Verbosity
 * -v - The task results are displayed
 * -vv - Taks Results and Task configuration
 * -vvv - Includes information about connections to managed hosts
 * -vvvv - Adds extra verbosity options to the connection plug-ins

#### Syntax Verificaiton
* Using **--syntax-check** options verifies the syntax of a playbook.  It is recommeded before running a playbook on a host

#### Executing a Dry Run
* **-C** option can be used to execute a *dry run* of the playbook.
* A dry run will report what changes would have occured if the playbook were executed


## Implementing Multiple Plays
* Writing a playbook that contains multiple plays is very straightforward
* Each play in th eplaybook is written as a top-level list item in the playbook

### Remote Users and Privilege Escalation in Plays
* Plays can use different remote users or privilege escalation settings for a play than what is specified by the defaults in teh config file.  These can be set in th eplay itself
#### User Attributes
* When a play is run on a managed host, the user account used for task executions is based on setting in the ansible.cfg file.
* The *remote_user* keyword defines which user will run the task
* If privilege e3sclation is enabled, other keywords such as *become_user* can also have an impact
* A *remote_user* keyword can be added within a play to override the *remote_user* described in the *ansible.cfg* file
#### Privilege Escalation Attributes
* The *become* boolean keyword can be used to enable or disable privilege escalation
* If privilege escalation is enabled, the *become_method* can be used to define the privilege escalation method (ex. sudo)
* With privilege escalation enabled, the *become_user* keyword can define the user account to use

### Finding Modules for Tasks
#### Module Documentation
* http://docs.ansible.com
* **ansible-doc -l** command can be used to list modules from a control node
* **ansible-doc [module name]** can be used to display detailed documentation for a module
* **-s** can be used to see an example of how to use th emodule in a playbook
#### Module Maintenance
* Ansible ships with modules and has an active upstream community
* the ansible-doc command can be used to see the status of different modules
* The *status* and *supported_by* fields recrd the development status and who maintains th emodule


