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
What happends when a task fails and what conditions cause a task to fail

### Managing Task Errors in Plays
* Ansible evaluates the return code of each task to determine wheter the task succeeded or failed.
* Normally, when a task fails Ansible immediately aborsts the rest of teh play on that host
* Exceptions can be made using a number of Ansible features that can be used to manage task errors

#### Ignoring Task Failure
* Using **ignore_errors** set to **yes** will allow the playbook to continue even if the task fails
```
- name: Latest version of notapkg is installed
  yum:
    name: notapkg
    state: latest
  ignore_errors: yes
```

#### Forcing Execution of Handlers after Task Failure
* Normally, handlers notified by earlier tasks are note executed if a task fails.
* Use the **force_handlers:  yes** keyword to force handlers to execute regardless of task status
```
---
- hosts: all
  force_handlers: yes
  tasks:
    - name: a task which always notifies its handler
      command: /bin/true
      notify: restart the database

    - name: a task which fails because the package doesn't exist
      yum:
        name: notapkg
        state: latest

  handlers:
    - name: restart the database
      service:
        name: mariadb
        state: restarted
```

#### Specifying Task Failure Conditions
* You can use the **failed_when** keyword on a task to specify which conditions indicate that the task has failed.
```
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    failed_when: "'Password missing' in command_result.stdout"
```
* The **fail** module can also be used to force a task failure
```
tasks:
  - name: Run user creation script
    shell: /usr/local/bin/create_users.sh
    register: command_result
    ignore_errors: yes

  - name: Report script failure
    fail:
      msg: "The password is missing in the output"
    when: "'Password missing' in command_result.stdout"
```
#### Specifying When a Task Reports "Changed" Results
* The **changed_when** keyword can be used to control when a task reports that it has changed.
```
  - name: get Kerberos credentials as "admin"
    shell: echo "{{ krb_admin_pass }}" | kinit -f admin
    changed_when: false
```
```
tasks:
  - shell:
      cmd: /usr/local/bin/upgrade-database
    register: command_result
    changed_when: "'Success' in command_result.stdout"
    notify:
      - restart_database

handlers:
  - name: restart_database
     service:
       name: mariadb
       state: restarted
```

#### Ansible Blocks and Error Handling
* **blocks** are clauses that logically group tasks and can be used to control how tasks are executed. 
* The below example shows blocking tasks to adhere to one **when** statement
```
- name: block example
  hosts: all
  tasks:
    - name: installing and configuring Yum versionlock plugin 
      block:
      - name: package needed by yum
        yum:
          name: yum-plugin-versionlock
          state: present
      - name: lock version of tzdata
        lineinfile:
          dest: /etc/yum/pluginconf.d/versionlock.list
          line: tzdata-2016j-1
          state: present
      when: ansible_distribution == "RedHat"
```

* Blocks also allow for error handling in combination with the rescue and always statements. If any task in a block fails, tasks in its rescue block are executed in order to recover. After the tasks in the block clause run, as well as the tasks in the rescue clause if there was a failure, then tasks in the always clause run. 
* To summarize:
  * **block**: Defines the main tasks to run.

  * **rescue**: Defines the tasks to run if the tasks defined in the block clause fail.

  * **always**: Defines the tasks that will always run independently of the success or failure of tasks defined in the block and rescue clauses. 

```
  tasks:
    - name: Upgrade DB
      block:
        - name: upgrade the database
          shell:
            cmd: /usr/local/lib/upgrade-database
      rescue:
        - name: revert the database upgrade
          shell:
            cmd: /usr/local/lib/revert-database
      always:
        - name: always restart the database
          service:
            name: mariadb
            state: restarted
```

