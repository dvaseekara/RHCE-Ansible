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
