# Chapter 4 - Managing Variables and Facts

## Managing Variables
### Introduction to Ansible Variables
* Variables providce a convenient way to manage dynamic values for an environment
* Examples:
	* users to create
	* Packages to install
	* Services to restart
	* Files to remove
	* Archives to retrieve from teh internet
### Naming Variables
* Variables must start with a letter and can only contain letters, numbers, and underscores

### Defining Variables
* Three basic scope levels for variables:
	1. Global Scope - Variables set from command line or Ansible config
	2. Play scope: Variables set in the play ad realated structures
	3. Host scope: Variables set on host groups and individual hosts by the inventory, fact gathering, or registered tasks

* If the same variable name is defined at more than one level , the level with the highest precedence wins.  
* A narrow scope takes precedence over a wider scope

### Variables in Playbooks
#### Defining Variables in Playbooks
* Playbook variables can be defined in multiple ways
1. Place a variable in a **vars** block at the beginning of a playbook
```
- hosts: all
  vars:  
    users: joe
	home:  /home/joe
```
2. Variables can be placed in external files.  Point ot the relative location of the external file from the playbook:
```
- hosts: all
  vars_files:
   - vars/users.yml
```
External File:
```
user: joe
home: /home/joe
```

### Using Variables in Playbooks

* After a variable has been declared the can be referenced by placing the variable name in double curly braces **({{}})**
```
vars:
  users: joe
  
tasks:
 - name:  Creates the user {{ user }}
   user:  
     name: "{{ user }}"
```

** Weird Ansible Caveat: When a variable is used as the first element to start a value, quotes are mandatory**

### Host Variables and Group Variables
* Variables that apply directly to hosts fall into two broad categories
 1. host variables - specifi to a particular host
 2. group variables - apply to all hosts in a host group or in a group of host groups
* Host variables take precedence over group variables
* Variables defined directly in playbook take precedence over booth
* You can define variables directly in the inventory file but its a silly way to do it (Don't do it)

#### Using Directories to Populate Host and Group Variables
* This is the preferred way to define variables for hosts and host groups
* Create two directories *group_vcars* and *host_vars* in the same directory as the inventory file 
* In the *group_vars* directory you can add a file for each group in your inventory to hold group level variables
* In the *host_vars* directory you can add a file for each host in your inventory to hold host specific variables

### Overriding Variables from teh Command Line
Inventory varialbes are overridden by playbook variables and Playbook variables can be overriden by command line variables like so:
```
ansible-playbook main.yml -e "package=apache"
```

### Using Arrays as Variables
* This is a pretty sweet way of cleaning up more complicated set of variables
* If organized this way you can browse through your variables
* Example:
```
user1_first_name: Tommy
user1_second_name: Chong
user1_home: /users/tchong
user2_first_name: Cheech
user2_second_name: Marin
user2_home: /users/cmarin
```
Can be rewritten as:
```
users:
  tchong:
    first_name: Tommy
    last_name: Chong
    home_dir: /users/tchong
  acook:
    first_name: Cheech
    last_name: Marin
    home_dir: /users/cmarin
```

You can then use these variables:
```
# Returns 'Tommy'
users.tchong.first_name

# Returns '/users/cmarin'
users.cmarin.home_dir
```

### Capturing Command Output with Registered Variables
* The *register* statement can be used to capture the output of a command into a temporary bariable that can be used later in the playbook
* Good way of capturing dynamic varialbes that may be generated earlier in the playbook

```
---
- name: Installs a package and prints the result
  hosts: all
  tasks:
  	- name: Install the package
	  yum:
	    name:  httpd
		state: installed
	  **register:  install_result**
	  
	-**debug: var=install_result**
```
* The **debug** modeule is used to dump to the terminal.  In this instance the playbook runs an install package and the reults are dumpted into the terminal




## Managing Secrets


## Managing Facts