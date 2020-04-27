# Chapter 3 - Implementing Playbooks

## Writing and Runnig Playbooks

### Ansible Playbooks and Ad Hoc Commands
* Ad Hoc commands can run sigle commands against a host but the real power of Ansible is running cmplex tasks using playbooks
* A *Play* is an ordered set of tasks.  A *playbook* is a text file containing one or more plays

### Formatting an Ansible Playbook

* consider the below snipped and how it can be rewritten as a play
'''
ansible -m user -a "name=newbie uid=4000 state=present" servera.lab.example.com
'''

'''
---
- name: Configure important user consistently
  hosts: servera.lab.example.com
  tasks:
    - name: newbie exists with UID 4000
	  user:
	    name: newbie
		uid: 4000
		state: present
'''

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

