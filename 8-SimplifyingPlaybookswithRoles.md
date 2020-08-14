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

### Time synchronization Role Example

### Selinux Role Example

## Creating Roles

## Deploying Roels with Ansible Galaxy
