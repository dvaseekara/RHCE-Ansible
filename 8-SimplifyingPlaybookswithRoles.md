# Chapter 8 - Simplifying Playbooks with Roles

## Describing Role Structure
* Ansible roles provide a way for you to make it easier to reuse Ansible code generically.
* it uses a standardized directory structure with all the tasks, variables, files, templates, and other resources needed to provision infrastructure or deply applications
* You can copy that role from project to project simply by copying the directory
* You can simply call that role from a play to execute it
* if written well, the role will allow you to pass variables that are usage specific
* Benefits:
  * Group content allowing eash sharing of code
  * Generic essential elements of a system type can be defined (web server, database server, git repo, etc)
  * make large projects manageable
  * Can be developed in parallel by different administrators
  
### Examining Ansible Role Structure
* Ansible roles all follow the same standardized directory structure
```
user.example/
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
```
* **defaults** - contains the default values of role variables that can be overwritten when the role is used.  Low precedence variables that are intended to be overwritten
* **files** - contains static fiels that are referenced by role tasks
* **handlers** - contains the role's handler definitions
* **meta** - contains information about the role (author, license, platforms, optional role dependencies)
* **tasks** - contains the role's task definitions
* **templates** - contains jinja2 templates that are referenced by role tasks
* **tests** - playbooks that can be used to test the role
* **vars** - Defines the role's variable values.  High precedence varialbes that are *not* intended to be changed.

### Defining Variables and Defaults
* Role variables are defined in the role/vars/main.yml file in key value pairs and can be referenced like any other variables:  `{{ variable_key }}`.
* These variables have high precedence and cannot be overwritten by inventory variables
* *Default variables* are defined in the role/default/main.yml and meant to be overwritten.

### Using Ansible Roles in a Playbook
```
---
- hosts: remote.example.com
  roles:
  - role1
  - role2
    var1: val1
	var2: val2
```
* For each role specified, role tasks, handlers, varaiables, and dependencies will be imported into the playbook, **in that order**
* File references in the role can be used withrout absolute or releative path names.  Ansible will look in the *files* directory of the role
* When you import roles into a play, they will run before any tasks defined in the playbook.

## Controlling Order of Execution
* Order execution for *role* tasks and handlers:
1. role tasks
2. play tasks
3. role handlers
4. play handlers

* You can add *pre_tasks* and *post_tasks* to define tasks that you need to run before any role tasks or after any handlers they notify.

* Roles can be added to a play using an ordinary task, not just by including them in the *roles* section of a play using *include_role* or *import_role*
```
- name: Execute a role as a task
  hosts: remote.example.com
  tasks:
    - name: A normal task
      debug:
        msg: 'first task'
    - name: A task to include role2 here
      include_role: role2
```

## Reusing Content with System Roles

### Red Hat Enterprise System Roles
* RHEL 7.4 and newer versions provided the following roles as part ot the *rhel-system-roles8 package

|**Name**|**State**|**Role Description**|
|--------|---------|--------------------|
|rhel-system-roles.kdump|Fully Supported|configures the kdump crash recovery services|
|rhel-system-roles.network|Fully Supported|Configures network interfaces|
|rhel-system-roles.selinux|Fully Supported| Configures and manages SELinux customization|
|rhel-system-roles.timesynce|Fully Supported|Configures time syunchronization usint Network Time Protocol|
|rhel-system-roles.postfix|Technology Preview|Configures each host as a Mail Transfer Agent using the Postfix service|
|rhel-system-roles.firewall|In development|Configures a hopst's firewall|
|rhel-system-roles.tuned|In development|configures the *tuned* service to tune system performance|

#### Simplified Configuration Management
* Roles simplify version differences between RHEL versions.  As an example, RHEL 7 uses chronyd while RHEL 6 uses *ntpd* but administrator no longer need to maintain both.  The System *rhel-system-roles.timesync* role will configure both
#### Support for RHEL System Roles
* RHEL System Roles are supported by Red hat.  Linux System Roles are not.

### Installing RHEL System Roles
```
yum install rhel-system-roles
```
* Once installed the roles can be found at `/usr/share/ansible/roles/`
* Each roles' documentation directory contains a *README.md* file with describes the role
* Documentation for upstream roles can be found at (https://galaxy.ansible.com)

### Role Examples:
* For brevity, I'm not going to describe these examples.  The *ReadMe* files in the role directory are more than enough.
* Role can be found in the `/usr/share/doc/rhel-system-roles/timesync` directory


## Creating Roles

### The Role Creation Process
1. Create the role directory structure
2. Define the role content
3. use the role in a playbook

### Creating the Role Directory Structure
* By default ansible looks for roles in a subdirectory called `roles` in the directory containingyour Ansible Playbook
* If Ansible cannot find the role there, it looks at the directories specified by the Ansible configuration setting `roles_path`
```
~/.ansible/roles:/usr/share/ansible/roles:/etc/ansible/roles
```
* This allows playbooks to share roles on your system
* Checkout the directory structure for roles above
#### Creating a role Skeleton
* The **ansible-galaxy** tool is used to manage Ansible roles, includint the creation of new roles
```
ansible-galaxy init my_new_role
ansible machine go brrrr
```
### Defining the Role Content
* A good place to start with role content is the `ROLENAME/tasks/main.yml`
#### Recommended Practices for Role Content Development
* Maintain each role in its own git repo
* Keep sensitive variables out of the role variables/playbooks - Ansible-vault it
* Use **ansible-galaxy init** to start the role.  Remove any directories you don't need!
* Create and maintain the README.md and meta/main.yml files you lazy dog
* Keep your role focused on a specific purpose or function
* Reuse and refactor roles often.  Resist creating new roles for edge configs.

### Defining Role Dependencies
* Role dependencies allow for inclusion of other roles to a role as dependencies.
* Dependencies are defined in meta/main.yml
```
---
dependencies:
- role: apache
  port: 8080
- role: postgres
  dbname: serverlist
  admin_user: felix
```
### Using the Role in a Playbook
* To access a role from a playbook use `roles:`
```
---
- name: Playbook to show roles
  hosts: all
  become: true
  remote_user: devops
  roles:
  - motd
```
* This scenario assumes that the motd role is in the `role` subdirectory in the same directory as the playbook

### Changing a Role's Behavior with Variables
* A well-written role will have default variables that can be manipulated to run different configurations of the role
* Default role varialbes can be overwritten by:
	1. an inventory file, either as a host variable or a group variable
	2. in a YAML file under the group_vars or host_vars directory of a playbook project
	3. as a variable nested in the *vars* keyword of a play
	4. as a variable when including the role in *roles* keyword of a play
	
## Deploying Roles with Ansible Galaxy
### Introducing Ansible Galaxy
* Public library of Ansible content written by a variety of Ansible administrators and users.
* It has a GUI, you go figure it out.
### Ansible Galaxy command Line Tool
* Can be used to search for, display information about, install, list, remove, or initialize roles!
```
ansible-galaxy search
# can filter by --author, --playform, --galaxy-tags

ansible-galaxy info
```
#### Installing Roles
```
ansible-galaxy install
```
*  By default roles will be installed to the first option in your *roles_path* or by env var *ANSIBLE_ROLES_PATH*
*  You can use the -p option to install it in a different directory
*  you can define roles needed for a playbook in a requirements.yml file and have ansible-galaxy install all the roles listed there:
*  you can even point to roles from a private repo
```
- src: geerlingguy.redis
  version: "1.5.0"
  
- src: https://gitlab.com/guardianproject-ops/ansible-nginx-acme.git
  scm: git
  version: 56e00a54
  name: nginx-acme
```
* Simply run:
```
ansible-galaxy install -r roles/requirements.yml -p roles
```
#### Managing Downloaded Roles
```
ansible-galaxy list
ansible-galaxy remove nginx-acme-ssh
```


