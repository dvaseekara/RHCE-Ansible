# Chapter 5 - Implementing Task Control

## Writing Loops and Conditional Tasks
### Task Iteration with Loops
* Ansible supports iterating a task over a set of items using the *loop* keyword

#### Simple Loops
* Iterates over a set of items using the *loop* keyword and adding a list of items over which the task should be iterated
* the *item* value holds the value used during each iterationn
* Example:  these two *service* modules can be replaced with a loop
```
- name: Postfix is running
  service:
    name: postfix
    state: started

- name: Dovecot is running
  service:
    name: dovecot
    state: started
```
LOOP:
```
- name: Postfix is running
  service:
    name:"{{ item }}"
	state: started
  loop:
    - postfix
	- dovecot
```
* The list used by the loop can be proviced by a variable:
```
vars:
  mail_services:
    - postfix
	- dovecot
tasks:
	- name: Postfix and Dovecot
	  service:
	    name:  "{{ item }}"
		state:  started
	  loop: "{{  mail_services }}
```

#### Loops over a List of Hashes or Dictionaries
* The loop list can be a hash or a dictionary 
```
- name:  User exist and are in the correct groups
  users:
    name:  "{{ item.name }}"
	state: present
	groups:  "{{ items.group}}
  loops:
   - name: jane
     groups: wheel
   - name: joe
     groups:  root
	 
```

#### Earlier-Style Loop Keywords
* Going to be depracated soon, not going to bother

#### Using Register Variables with Loops
* The *register* keyword can also capture the output of a task that loops
* The output will be an array of the results from an iterated tasks

### Running Tasks Conditionally
* Examples of scenarios where conditionals can be used to evaluate execution
1. A hard limit can be defined in a variable and compared against the available memory on a managed host
2. The output of a command can be captured and evaluated by Ansible to determine whether or not a task completed before taking further action
3. Use ansible facts to determine the managed host nbetwork config and decide which template file to send
4. The number of CPUs can be evaluated ro properly tune a web server
5. Compare a registered variable with a predefined variable to determine if a service changed

#### Condittional Task Syntax
* The **when** varialbe is used to introduce a conditional to a task
```
---
- name: Simplle Boole Task
  hosts:  all
  vars:
    run_my_task: true
	
  tasks:
    - name: httpd package is installed
	  yum:
	    name: httpd
	  when:  run_my_task
```

#### Example Conditionals:
|Operation|	Example|
|---------|--------|
|Equal (value is a string)|	ansible_machine == "x86_64"|
|Equal (value is numeric)|	max_memory == 512|
|Less than|	min_memory < 128|
|Greater than|	min_memory > 256|
|Less than or equal to|	min_memory <= 256|
|Greater than or equal to|	min_memory >= 512|
|Not equal to|	min_memory != 512|
|Variable exists|	min_memory is defined|
|Variable does not exist|	min_memory is not defined|
|Boolean variable is true. The values of 1, True, or yes evaluate to true.| 	memory_available|
|Boolean variable is false. The values of 0, False, or no evaluate to false.| not memory_available|
|First variable's value is present as a value in second variable's list|	ansible_distribution in supported_distros|

#### Testing Multiple Conditions
* One *when* keyword can be used to evaluate multiple conditionals.  Use the **and** or **or** keywords grouped with parentheses
* multiple *and* conditionals can be expressed as a list:
```
when:
  - ansible_distribution == "7.5"
  - ansible_kernel == "3.10"
```

### Combininb Loops and Conditional Tasks
Examples:
```
- name: install mariadb-server if enough space on root
  yum:
    name: mariadb-server
    state: latest
  loop: "{{ ansible_mounts }}"
  when: item.mount == "/" and item.size_available > 300000000
```
Example:  Restarting httpd by evaluating the result of the Get Postfix server status
```
---
- name: Restart HTTPD if Postfix is Running
  hosts: all
  tasks:
    - name: Get Postfix server status
      command: /usr/bin/systemctl is-active postfix 1
      ignore_errors: yes2
      register: result3

    - name: Restart Apache HTTPD based on Postfix status
      service:
        name: httpd
        state: restarted
      when: result.rc == 04
```

## Implementing Handlers
* Handlers enables certain tasks to only run when another task changes the managed host
### Ansible Handlers
* Handlers are tasks that respond to a notification triggered by other tasks
* Tasks only notify their handlers when the task changes something on a managed host
* Each handler has a blobally unique name and is triggered at the end of a block of tasks in a playbook
* if not task notifies the handler by name then the handler will not run
* If more than one task notify the handler, the handler will run exactly once after all other tasks in the play have completed
* Because Handlers are tasks they can run the same modules as tasks.
* Commonly handlers are used to rebbot hosts and restart services

Example (Note the **notify** and **handlers** keywords):
```
tasks:
  - name: copy demo.example.conf configuration template
    template:
      src: /var/lib/templates/demo.example.conf.template
      dest: /etc/httpd/conf.d/demo.example.conf
    notify:
      - restart apache

handlers:
  - name: restart apache
    service:
      name: httpd
      state: restarted
```

### Benefits of Handlers
* Handlers always run in the order specified by the handlers section of the play.  They do not run in order of the *notify*
* Handlers normally run after all other tasks in the play are complete
* Handler names exsist in a per-play namespace.  If two handlers are incorrectly named the same on ly one will run
* Even if more than one task notifies a handler, the handler only runs once
* If a task that includes a *notify* statement does not report a *changed* result, the handler is not notified

## Handling Task Failure

