# Chapter 6 - Deploying Files to Managed Hosts

## Modifying and Copying Files to Hosts


## Deploying Custom Files with Jinja2 Templates
### Describing Files Modules
* The *Files* modules library includes tasks realted to most Linux file management functions.
|**Module Name**	|**Module Description|
|blockinfile		|Insert, update, or remove a block of multiline text surrounded by customizable marker lines|
|copy				|Copy a file from the local or remote machine to a location on a managed host|
|fetch				|Works like copy but in reverse.  Fetches files from remote machines to the control node|
|file				|Does a lot:  permissions, ownership, SELinux contexts, timestamps, symlinks, hard links, directories, etc|
|lineinfile			|Ensure that a particular line is in a file. Replace an existing line.  Use when editing a single line in a file|
|stat				|Retrieve status information for a file, similar to the Linux stat command|
|synchronize		|A wrapper around the rsync command|


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









