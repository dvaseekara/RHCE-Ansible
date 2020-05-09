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
### Introducing Ansible Vault
* Sensitive data cannot be stored in plaintext files for obvious reasons
* Ansible comes with **Ansible Vault** and can encrypt and decrypt any structured daat file used by Ansible
* A command line tool named **ansible-vault** is used to create, edit, encrypt, decrypt, and view fiels
* Ansible Vault can encrypt data including inventory varialbes, variablesfiles in a playbook, variables passed as an argument when executing the playbook, or variables defined in Ansible roles.

#### Creating an Encrypted File
* To reate a new encrypted file use teh command below.  After entering the password you should be able to edit the file using *vim*.  To use a different editor update the *EDITOR* env variable to a different selecting
```
ansible-vault create secret.yml
New Vault password: redhat
Confirm New Vault Password:  redhat
```
* You can also pass the vault password through a vault password file.  You need to carfully protect theis file using file permissions
```
ansible-vault create --vault-password-file=vault-pass secret.yml\
```
#### Viewing an encrypted vault file
```
ansible-vault view secret.yml
```
#### Editing an encrypted vault file
* The below command decrpyts the file to a temporary file and allow you to edit it.  When saved the tmp file is removed.
```
ansible-vault edit secret.yml
```
#### Decrypting an Existing File
```
ansible-vault decrypt secret1.yml --output=secret1-decrypted.yml
```
#### Changing the password on an encrypted file
```
ansible-vault rekey secret.yml
```
When using a vault password file:
```
ansible-vault rekey --new-vault-password-file=NEW_VAULT_PASSWORD_FILE secret.yml
```

### Playbooks and Ansible Vault
* To run a playbook that uses an ansible vault encrypted file you must provide the enccryption password like so:
```
ansible-playbook --vault-id @prompt site.yml
Vault password (default): redhat
```
* You can also use **--vault-password-file** option
```
ansible-playbook --vault-password-file=vault-pw-file site.yml
```

#### Recommended Practices for Variable File Management
* Sensitive varialbes and non sensitive varialbes are kept in separate files
* Files containing sensitive data should be protected with ansible-vault
* files that contain group variables and host variables should go in the host_vars or group_vars directories
* Sensitive variables can be added to these directories as ansible-vault encrypted files as well
* If you are using multiple vault passwords with your playbook, make sure that each encrypted file is assigned a vault ID, and that you enter the matching password with that vault ID when running the playbook.



## Managing Facts
* Facts are varialbes that are located on the managed host machine and can be discovered by Ansible
* Every play runs the *setup* module before the first task to gather facts
* The *Gathering Facts* task in ansible reflects this.  You don't have to explictly run this task, it is run automatically.
* You can use the *ansible_facts* variable to dump gathered facts onto the terminal for inspection:
```
- name: Fact dum;
  hosts: all
  tasks:
    - name: Pring all facts
      debug:
	    var: ansible_facts
```
* When a fact is used in a playbook, asible dynamically substitutes the varialbe name for the fact
```
---
- hosts: all
  tasks:
  - name: Prings various Ansible facts
    debug: 
	  msg: >
	    The default IPV4 address of {{ ansible_facts.fqdn }}
		is {{ ansible_facts.default_ipv4.address }}
```
*Commonly used facts:
|Fact										| 	Variable													|
|-------------------------------------------|---------------------------------------------------------------|
|Short host name							|	ansible_facts['hostname']									|
|Fully qualified domain name				|	ansible_facts['fqdn']										|
|Main IPv4 address (based on routing)		|	ansible_facts['default_ipv4']['address']					|	
|List of the names of all network interfaces|	ansible_facts['interfaces']									|
|Size of the /dev/vda1 disk partition		| 	ansible_facts['devices']['vda']['partitions']['vda1']['size']|
|List of DNS servers						|	ansible_facts['dns']['nameservers']							|
|Version of the currently running kernel	|	ansible_facts['kernel'] 									|

### Ansible Facts injected as Variables
* Old way of doing things; Don't need to learn

### Turn off Fact Gathering
* Simply set the *gather_facts* variable to no like so:
```
---
- name: This play gathers no facts
  host: da_host
  gather_facts: no
```

### Creating Custom Facts
* Along with the standard facts listed above you can careate custom facts that live on managed hosts
* Custom facts can be defined in a static INI or JSON file.  They can also be a script that generates a JSON output (dynamic)
* By default *setup* module loads custom facts from files and scripts in each managed hosts's **/etc/ansible/facts.d** directory
* the name of each file must end in **.fact**

* Custom facts are stored by *setup* module in teh ansible_facts.ansible_local variable.
* Facts are organized based on the name of the file
* Example: 
	* file name on managed host: /etc/ansible/facts.d/custom.fact
	* Retrieve syntax:  ansible_facts.ansible_local['custom']['user']
	
### Using Magic Variables
* **hostvars**
* **group_names**
* **groups**
* **inventory_hostname**

	