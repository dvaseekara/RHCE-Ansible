# Chapter 6 - Deploying Files to Managed Hosts

## Modifying and Copying Files to Hosts
### Describing Files Modules
* The *Files* modules library includes tasks realted to most Linux file management functions.


| **Module Name**	| **Module Description** |
| --------------------- | ---------------------- |
| blockinfile		| Insert, update, or remove a block of multiline text surrounded by customizable marker lines |
| copy				| Copy a file from the local or remote machine to a location on a managed host |
| fetch				| Works like copy but in reverse.  Fetches files from remote machines to the control node |
| file				| Does a lot:  permissions, ownership, SELinux contexts, timestamps, symlinks, hard links, directories, etc |
| lineinfile			| Ensure that a particular line is in a file. Replace an existing line.  Use when editing a single line in a file | 
| stat				| Retrieve status information for a file, similar to the Linux stat command |
| synchronize		| A wrapper around the rsync command |


### Automation Examples with Files Modules
#### Ensuring a File Exists on Managed Hosts
* The below task works like the *touch* command in Linux.  It creates the file if it doesn't exist or updates the modificaiton time if it does.  This example also ensures that the owning user, group, and permissions are set to specific values:
```
- name: Touch a file and set permissions
  file:
    path:  /path/to/file
	owner:  user1
	group:  group1
	mode:  0640
	state:  touch
```

#### Modifying File Attributes
* The following task ensures that th eSELinux context type attribute of the *samba_file* is *samba_stare_t*.  The behavior is similar to **chcon**
```
- name:  SELinux type is set to samba_share_t
  file:
    path:  /path/to/samba_file
	setype: sambe_share_t
```

#### Making SELinux File Context Changes Persistent
* Throw back to RHCSA - using **chcon** to set file context is temporary unless you run **semanage fcontext**.  I can be reversed using **restorecon**.
* Similarly using *file* module's **setype** is temporary.  Use **sefcontext** to make it persistent
```
- name: SELinux type is persistently set to samba_share_t
  sefcontext:
    target: /path/to/samba_file
    setype: samba_share_t
    state: present
```

#### Copying and Editing Files on Managed Hosts
* user *force: yes* or *force: no* to overwrite the existing file on the manged host
```
- name: Copy a file to managed hosts
  copy:
    src: file
	dest: /path/to/file
```

* use the *fetch* module to retrieve files from managed hosts
```
- name: Retrieve SSH key from reference host
  fetch:
    src: "/home/{{ user }}/.ssh/id_rsa.pub
    dest: "files/keys/{{ user }}.pub"
```

* Ensure a single line of text exists in an existing file use *lineinfile*
```
- name: Add a line of text to a file
  lineinfile:
    path: /path/to/file
    line: 'Add this line to the file'
    state: present
```

* Add a block of text to exsiting file using *blockinfile*
```
- name: Add additional lines to a file
  blockinfile:
    path: /path/to/file
    block: |
      First line in the additional block of text
      Second line in the additional block of text
    state: present
```

#### Removing a File from Managed Hosts
* Make sure a file doesn't exist in the managed host:
```
- name: Make sure a file does not exist on managed hosts
  file:
    dest: /path/to/file
    state: absent
```

#### Retrieving the Status of a File on Managed Hosts
* The *stat* module retrifeves facts for a file.

```
- name: Verify the checksum of a file
  stat:
    path: /path/to/file
    checksum_algorithm: md5
  register: result

- debug
    msg: "The checksum of the file is {{ result.stat.checksum }}"
```
* Use **ansible-doc** to learn more about the *stat* command

#### Synchronizing Files Between the Control Node and Managed Hosts
```
- name: synchronize local file to remote files
  synchronize:
    src: file
    dest: /path/to/file
```

* Again use **ansible-doc** to get more details on how each of the above tools work

## Deploying Custom Files with Jinja2 Templates
### Templating Files
* Ansible allows editing of file usintg *lineinfile* and *blockinfile* but using templats is much more effective

### Intro to Jinja2
* Ansible uses the Jinja2 templating system for template files.
* Jinja2 two syntax can be used reference variables in playbooks too

#### Using Delimiters
* Jinja2 uses {% EXPR %} for expressions or logic suchs as loops and {{ EXPR }} for variables
* {# COMMENT #} can be used to enclose comments that shouldn't appear in the final file

### Building a Jinja2 template
* A Jinja2 template can consist of variables, facts, and expressions that will be rendered. Variables can be specified in teh vars section of the playbook 
* Example:
```
# {{ ansible_managed }}
# DO NOT MAKE LOCAL MODIFICATIONS TO THIS FILE AS THEY WILL BE LOST

Port {{ ssh_port }}
ListenAddress {{ ansible_facts['default_ipv4']['address'] }}

HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key

SyslogFacility AUTHPRIV

PermitRootLogin {{ root_allowed }}
AllowGroups {{ groups_allowed }}

AuthorizedKeysFile /etc/.rht_authorized_keys .ssh/authorized_keys

PasswordAuthentication {{ passwords_allowed }}

ChallengeResponseAuthentication no

GSSAPIAuthentication yes
GSSAPICleanupCredentials no

UsePAM yes

X11Forwarding yes
UsePrivilegeSeparation sandbox

AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS

Subsystem sftp	/usr/libexec/openssh/sftp-server
```

### Deploying Jinja2 Templates
* The *template* module can be used to deploy Jinja2 Templates
* The *src* key specifies the source jinja2 tempalte and the *dest* key specifies where the file will be created

```
tasks:
  - name: template render
    template:
      src: /tmp/j2-template.j2
      dest: /tmp/dest-config-file.txt
```

### Managing Templated Files
* Its good practice to use *ansible_managed* to indicate that a file is managed by a jinja2 template and shouldn't be manually updated

### Control Structures
#### Using Loops
```
{% for user in users %}
	{{ user }}
{% endfor % }
```
```
{# for statement #}
{% for myuser in users if not myuser == "root" %}
User number {{ loop.index }} - {{ myuser }}
{% endfor %}
```
```
{% for myhost in groups['myhosts'] %}
{{ myhost }}
{% endfor %}
```

#### Using Conditionals
*  In the following example, the value of the result variable is placed in the deployed file only if the value of the finished variable is True. 
```
{% if finished %}
{{ result }}
{% endif %}
```

### Variable Filters
* Jinja2 provides filters which change the output format for template expressions.
* Filters are available for languages such as YAML and JSON

```
{{ output | to_json }}
{{ outout | to_yaml }}

{{ output | to_nice_json }}
{{ output | to_nice_yaml }}

{{ output | from_json }}
{{ output | from_yaml }}
```

### Variable Tests
 The expressions used with when clauses in Ansible Playbooks are Jinja2 expressions. Built-in Ansible tests used to test return values include failed, changed, succeeded, and skipped. The following task shows how tests can be used inside of conditional expressions.
```
tasks:
...output omitted...
  - debug: msg="the execution was aborted"
    when: returnvalue is failed
```







