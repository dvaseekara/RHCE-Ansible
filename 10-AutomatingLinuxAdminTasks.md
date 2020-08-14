# Chapter 10 - Automating Linux Administration Tasks
## Managing Software and Subscriptions
* Subscribe systems, configure software channels and repositories, and manage RPM packages on mangesd hosts

### Managing Packges with Ansible
* The *yum* Ansible module uses the *Yum Package Manager*

#### Gathering Facts about installed packages
* The *package_facts* module gathers package facts and puts them in *ansible_facts.packages*

### Registering and Managing Systems with Red Hat Subscription Management
#### Registering and SubscribingNew systems
* *redhat_subscription* and *rhsm_repository* modules are provided to manage RHEL subscriptions
```
- name: Register and subscribe the system
  redhat_subscription:
    username: user
	password: pass
	pool_ids: poolID
	state: present
```
#### Enable Red Hat Software Reepositories
```
- name: Enable Red Hat Repositories
  rhsm_repository:
    name:
	- rhel-8-for-x86_64-baseos-rpms
	- rhel-9-for-x8
	state: present
```

### Configuring a Yum Repository
#### Declaring a Yum Repository
```
---
- name: Configure the company Yum repositories
  hosts: servera.lab.example.com
  tasks:
  - name: Ensure Example repo exists
    yum_repository:
	  file: example
	  name: example-internal
	  description;  Example Inc. Internal Yum repo
	  baseurl: http://materials.example.com/yum/repository
	  enabled: True
	  gpgcheck: yes
	  state: present
```
* use *rpm_key* to import an rpm key
```
- name: Deploy the GPG public key
  rpm_key:
    key:  http;//asdfasdf.com/
	state present
```

## Managing Users and Authentication
### The User Module
* you can manage a number of parameters including remove user, set home directory, set UID of system accounts, manage passwords, and associated groupings
* You need to provide a hashed password for the password parameter in order to create a user that can log into the machine
* Example
```
- name: Add new user to the development machine and assign appropriate groups
  user:
    name: devops_user
	shell: /bin/bash
	groups: sys_admins, developers
	append: yes
```
* If you don't use append the groups will overwrite in place
* you can also specify *generate_ssh_key* 
```
- name: user with SSH key
  user: 
    name: user
	generate_ssh_key: yes
	ssh_key_bits: 2048
	ssh_key_file: .ssh/id_my_rsa
```

### The Group Module
* manage (add, delete, modify) groups on the managed hosts
```
- name: Verify that auditors group exists
  group:
	name: auditors
	state present
```
### The Known Hosts Module
* If you have a large number of host keys to manage you will want to use the *known_hosts* module to add or remove host keys
```
- name: copy host keys to remote servers
  known_hosts:
    path: /etc/ssh/ssh_known_hosts
    name: user1
    key: "{{ lookup('file', 'pubkeys/user1') }}"1
```

### The Authorized Key Module
* The authorized_key module allows you to add or remove SSH authorized keys per user accounts. When adding and subtracting users to a large bank of servers, you need to be able to manage ssh keys. 
```
- name: Set authorized key
  suthorized_key
    user: user1
	state: present
	key: "{{ lookup('file', '/home/user1/.ssh/id_rsa.pub') }}
```

## Managing the Boot Process and Scheduled Processes
### Scheduling with the at Module
* Quick on-time scheduling can be done with the *at* module
```
- name: remove tempuser
  at:
    command: userdel -r tempuser
	count: 20
	units: minutes
	unique: yes
```

### Appending with the cron Module
* The cron module will append commands directly into the crontab of the user you designate
```
- name: cron module example
  cron:
    name: "Flush Bolt"
	user: "root"
	minute: 45
	hour: 11
	job: "php ./app/nut cache:clear"
```
### Managing Services with the systemd and service Modules
* Youve seen snough of the service module
* systed just does what servcie does in different words

### The Reboot Module
```
- name: "Reboot after patching"
  reboot:
    reboot_timeout: 180

- name: force a quick reboot
  reboot:
```

### The Shell and Command module

## Managing Storage
* Partition storage devices, configure LVM, format partitions or logical volumes, mount file systems and add swap files or spaces

### Configure Storage with Ansible Modules
* Partitiion the Devices
* Create Logical Volumes
* Create and Mount file systems

#### The *parted* Module
* Allows partitioning of block devices with a specified size, flag, and alignment
| Parameter name|Description|
|-----------|-------|
| align		| Configures partition alignment|
| device	| block Device|
| flags		| Flags for the partition|
| number 	| The partition number|
| part_end	| Partition size from teh beginning of the disk specified in parted supported units|
| state		| Creates or removes the partition
| unit 		| Size units for the partition information|

```
- name: New 10gb partition
  parted:
    device: /dev/vdb
	number: 1
	state: present
	part_end: 10gb
```

#### The *lvg* and *lvol* Modules
* Supports creation of logical volumes, including configuration of physical volumes, and volumen groups.
##### lvg
* Takes as parameter: block devices to configure as back end physical volumes for the volume group

| Parameter name	|Description|
|-------------------|-----------|
|pesize				| Physical extent size	|
|pvs				| comma separated devices to be configured as physical volumes for the volume group	|
|vg					| The name of the Volume Group	|
|state				| Creates or removes the volume	|

```
- name: Creates a volume Group
  lvg:
    pvs: /dev/vda, /dev/vdb
	vg: vg1
	pesize: 32
```
* Remember that Ansible is state based which means you can resize the volume group by add subsequent task that adds a physical volume using *lvg*

##### lvol
* Creates logical volumes and supports resizing, shrinking of those volumes and the filesystems on top of them.
|	Parameter name	| Description |
| lv				| The name of the logical volume |
| resizefs			| Resizes the filesystem with the logical volume	|
| shrink			| enable logical volume shrink						|
| size				| size of the logical volume						|
| snapshot			| The name of the snapshot for the logical volume	|
| state				| create or remove the logical volume				|
| vg				| The parent volume group for the logical volume	|


##### The *filesystem* Module
* Supports creating and resizing a filesystem
* Supports filesystem resizing for ext2, ext3, ext4, ext4dev, f2fs, lvm, xfs, and vfat

|	Parameter name	|	Description	|
| dev	| Block Device Name	|
| fstype |filesystem type	|
| resizefs	| Grows the filesystem size to the size of the block device

```
- name: Create an XFS filesystem
  filesystem:
    fstype: xfs
	dev: /dev/vdb1
```

##### The *mount* Module
* supports the configuration of mount points on /etc/fstab
| Parameter name	| Description	|
|-------------------|---------------|
| fstype			| filesystem type	|
| opts				| Mount options		|
| path				| Mount point path	|
| src				| Device to be mounted	|
| state				| Specify the mount status (mount/unmount)|

#### Configuring swap with Modules
* Ansible doesn't currently include modules to manage swap memory
* To add swap memoryu:
	1. Create new volume group
	2. Ceare new logical volume
	3. Run *mkswap* using command line
	4. Run *swapon* using command line

### Ansible Facts for Storage Management
```
ansible webservers -m setup -a 'filter=ansible_devices'
```