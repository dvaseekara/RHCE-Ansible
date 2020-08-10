# Chapter 7 - Managing Large Projects
## Selecting Hosts with Host Patterns

### Referencing Inventory Hosts
* Host partterns are used to specify the hosts to target
* Multiple form to get the host information are available
* Name of a managed host or a host group in the inventory, *hosts* field in a playbook
* Using patterns to manage what hosts a play targets can take away complexity from the playbook

#### Managed Hosts
* Name for a single managed host listed in the inventory.
* When the playbook runs the first play will run *Gathering Facts* on all managed hosts.  Failure during this task can cause the host to be removed from the play
* If an IP address is not listed in teh inventory then you cannot use it specify the host in a play

#### Specifying Hosts Using a Group
* When a group name is used as a host pattern, ansible will act on the hosts that are members of that group
* *all* is a special group including all hosts
* *ungrouped* is a special group that includes all hosts not in any group

#### Wildcards, lists, and logic
* using "*": 127.12.12.* or servera.*.example.com
* Using lists: servera, serverb
* Logic: servera, &serverb
	* will resolve to any hosts that are in servera and serverb
* Logic: datacenter1, !servera
	* will resolve to all hosts in datacenter1 except servera
	* you can do this with *all* and !groups too
	
## Managing Dynamic Inventories
### Generating Inventories Dynamically
* When working in an environment where machines come and go quickly a static inventory can't keep up
* Most IT environments will have systems that keep trak of which hosts are available
	* External Directory Service: Zabbix, FreeIPA, Active Directory servers
	* Installation Servers: Cobble
	* Baremetal management services: Red Hat Satellinte
	* Cloud Services: Amazan EC2, OpenStack deployment, etc etc
* Ansible supports *dynamic inventory* scripts that retrieve information from these sources when Ansible executes
* These scripts collect information from external sources and output the inventory in JSON
* *Dynamic inventory is specified just like a static inventory in the ansible.cfg or using -i*
* If the inventory file is an executable the it is treated as dynamic

### Contributed Scripts
* [Git Repo](https://github.com/ansible/tree/devel/contrib/inventory) contains a bunch of open source scripts but they are not supported

### Writing Dynamic Inventory Programs
* A custom dynamic inventory program can be written in any programming language as long as it returns info in a JSON format
* The **ansible-inventory** command can be a helpful tool
```
ansible-inventory -i inventoryfile --list
```
* More details on writing dynamic inventory script: [Ansible Devloper Documents](https://docs.ansible.com/ansible/latest/dev_guide/index.html)
* Start the script with an appropriate interpreter line (ex. #!/usr/bin/python)
* when given the --list option the script must pring a JSON-encoded hash/dict of all the hosts in the inventory
* when given the --host managed-host option the script must pring variables associted with that host in an empty JSON hash/dict

### Managing Multiple Inventories
* Ansible allows you to use multiple inventories in the same run
* Point to a directory and all the files in the directory can be included as inventory files (static or dynamic)

## Configuring Parallelism
### Configure Parallenlism in Ansible Using Forks
* When processing a playbook, Ansible runs ieach play in order.  Ansible also runs through each task in order.  
* All hosts must successfully complete a task before any host starts the next task in the play
* Ansible could simultaneously connect to all hosts in the play for each task but this could get stress the control node for larger environments
* The maximum number of simultaneous connections that Ansible makes is controlled by the *forks* parameter in tehe *ansible.cfg* file.  Defaults to 5
* Example
```
if you have 5 forks and 10 hosts: 
1. Ansible will run the first task on the first 5 hosts
2. run a second set of the first task on the next 5 hosts
3. repeat the process for the next task.
```

* You can tweak your fork load based on whether your plays run heavily on the managed host or on the control node.  For example, Linux administration runs heavily on the managed hosts so you can crank up the forks where as network or router swtich plays rely on the control node so be judicious with your forks
* you can use *-f* in the command line to override the default forks

### Managing Rolling Updates
* running al tasks on all hosts cal lead ot undesirable behavior.
* For example, if an play updates a cluster of load balanced web servers, it might need to take each web server out of service during the update.  If all servers are updated at the same time they could all be out of service
* The *serial* parameter could be used to run the hosts through plays in batches.  Each batch of hosts will run through the entire play before the next batch is run
* Remember, normally, Ansible runs all hosts throuh a task before moving on to the next text
* The *serial* keyword can be specified as a percentage or a number


## Including and Importing Files
### Including or Importing Files
* when you include content, it is a *dynamic* operation.  Ansible processes included content during the rufn of the playbook as content it reached
* when you import content, it is *static* operation. Ansible preprocesses imported content when the playbook is initially parsed, before the run starts

### Importing Playbooks
* *import_playbook* parameter can be used at the play level in a playbook to import external files that contain plays
* you can have a master playbook that imports additinoal playbooks.  If you import multiple playbooks they will be run in the order that you import them
* Example:
```
- name: Prepare the web server
  import_playbook: web.yml
  
- name: Prepare teh db server
  import_playbook: db.yml
```

### Importing and Including Tasks
```
*Task File*:
- name: Installs the httpd package
  yum: 
    name: httpd
	state: latest
- name: Starts the httpd service
  service:
    name: httpd
	state: started
```

#### Importing Task Files
* You can statically import a task file into a play by using *import_task* parameter under the tasks level
```
---
- name: Install web server
  hosts: webservers
  tasks:
  - import_tasks: webserver_tasks.yml
```
* *Importing* a task file means that tasks are inserted whtn the playbook is parsed and not when it is run
	* Conditional statements set on the import, such as *when*, are applied to each of the tasks that are imported
	* You cannot use loops
	* You cannot use a host or group inventory variable

#### Including Task Files
* You can dynamically *include* a task file int a play inside a playbook using *include_tasks*
```
---
- name: Install web server
  hosts: webservers
  tasks:
  - include_tasks: webserver_tasks.yml
```
* *Including* a does not process content in the playbook until the play is running and that part of the play is reached
	* Conditional statements such as *when* set o n the include determine whether or not the tasks are included in the play at all
	* **ansible-playbook --list-tasks** will not show included tasks.  It will show the file.  
	* You cannot use **ansible-playbook --start-at-task**
	* You cannot use *notify* to trigger a handler that is in an included task file.  You can trigger a handler in the main playbook.
	
#### Definig Variables for External Plays and Tasks
* Use variables to make your play and task files useful for different playbooks
Examples:
```
---
  - name: Install the {{ package }} package
    yum:
      name: "{{ package }}"
      state: latest
  - name: Start the {{ service }} service
    service:
      name: "{{ service }}"
      enabled: true
      state: started
```
```
...output omitted...
  tasks:
    - name: Import task file and set variables
      import_tasks: task.yml
      vars:
        package: httpd
        service: service
```
* For playbooks:
```
...output omitted...
- name: Import play file and set the variable
  import_playbook: play.yml
  vars:
    package: mariadb
```


