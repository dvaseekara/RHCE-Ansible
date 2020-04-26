# REDHAT SYSTEM ADMINISTRATION III: LINUX AUTOMATIN WITH ANSIBLE

Automate, provision, configure, and orchestrate Linux Environments
Learn how to install and configure Ansible on a management workstation and prepare managed hosts for automation
Write Ansible playbooks to automate tasks

# Chapter 1: Introducing Ansible

## Automating Linux Administration with Ansible

### Infrastructure as Code
* Be able to use machine-readable automation language to define and escribe the state you want your IT infrastructure to be in.
* Ideally this language must be human readable as well
* Manage in Version Control

### Mitigating Human Error
* Reduce the tasks performed manually on servers
* make changes rhrough updating automation code, rather than manually applying them to your servers
* Version Control allows for code and peer reviews.  And Documentation

### What is Ansible?
* Simple automation language that can perfectly describe an IT application infrastructure in playbooks
* Manage powerful automation tasks and adapt to many different workflows and environments
* Ansible is Simple
* Ansible is Powerful
* Ansible is Agentless
	* Ansible is built around an agentless architecture
	* Ansible runs tasks on hosts using openSSH or WinRM buy pushing out small programs called Ansible modules
	* The modules put the system in a specific state and then removed when Ansible is done.
	* No Agents means no additional custom security infrastructure
	* Important strengths
		* Cross platform support (Linux, windows, UNIX, network devices, VM, Cloud, etc)
		* Human Readable automation - Yaml Files baby
		* Perfect description and documentation of applications
		* Easy to manage in verstion control
		* Support dynamic inventories
		* Orchestration that integrates easily with other systems (Puppet, Jenkins, Red Hat Satellite, etc)

### Ansible Concepts and Architecture
* There are two tpyes of machines in Ansible architecture
	* Control Nodes
		* Ansible is installed and run from a control node
		* This machine also has copies of your Ansible project Files
		* Can be an administrator's laptop, a system shared by admins, or a server running Red Hat Ansible Tower
	* Managed Hosts
		* Listed in an inventory
* Instead of writing complex scripts, Ansible users create high-level plays to ensure a host or group of hosts re in a particular state
	* A play performs tasks on the host and is expressed in YAML format. 
	* A file that contains one or more plays is called a playbook
* Each task runs a module (a small piece of code (Python, PowerShell, or some other language) with specific arguments
	* Ansible ships with a wide variety of modules
* Playbooks are idempotent - You can safely run a playbook on teh same host multiple times
* Ansible also uses plug-ins (code that you can add to Ansible to extend/adampt it)
* *Red Hat Ansible Tower* is an enterprise framework to help you control, secure, and manage your Ansible automation at scale.
	* Provides an UI and RESTfulAPI - Not a core part of Ansible but a separate product
	
### The Ansible Way
* Complexity Kills Productivity
* Optimize for Readability
* Think Declaratively (Desired State rather than scripts)
* Use Cases
	* configuration Management
	* Application Deployment
	* Provisioning
	* Continuous Delivery
	* Security and Compliance
	* Orchestration

## Installing Ansible

### Control Nodes
* Ansible only needs to be installed on Control nodes from which it will be run
* Managed hosts do not need to have Ansible installed
* Control node should be a *LINUX or UNIX system* 
* *Python 3* or 2.7 or later needs to be installed in the control node
* *If you are running RHEL8, Ansible 2.8 can automatically use the platform-python package*
* [Downloading and Installing RH Ansible Engine] https://access.redhat.com/articles/3174981

#### Installation Procedure
1. Register your system to Red Hat Subscription Manager
```
subscription-manager register
```
2. Set a role for your system
```
subscription-manager role --set="Red Hat Enterprise Linux Server"
```
3. Attach your Red Hat Ansible Engine subscription
```
subscription-manager list --available
```
4. Use the pool ID of the subscription to attache the pool to the system
```
subscription-manager attache --pool=>engine-subscription-pool>
```
5. Enable RH Ansible Enbine Repo
```
subscription-manager repos --enable ansible-2-for-rhel-8-x86_64-rpms
```
6. Install Red Hat Ansible Engine
```
yum install ansible
```

### Managed Hosts
* Managed Hosts don't need to have an agent installed
* the control node connects to managed hosts (ssh) and makes sure its in the specified state
* Unix and Linux managed hosts need to have *Python 2.6+ or Python 3.5+* installed for most modules
```
yum install python36
```
* *Microsoft Windows-based Managed Hosts*
	* Most modules require *PowerShell 3.0+*
	* *.NET Framework 4.0+* is also requried
* *Managed Network Devices* - Router, Switches, etc
	* Because network devices cannot run python, ansible runs network modules on the control node and uses special connection methods to communicate with network devices
	* *Not part of this course* - woohoo!
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
	
		

** 