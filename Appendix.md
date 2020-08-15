### Review from RHCSA - Basic Storage
#### Partitionaing a disk
1. mklabel
2. mkpart
	a. size
	b. filesystem (xfs, ext2, ext4...)
3. udevadm settle
4. mount or /etc/fstab
5. systemctl daemon-reload

#### Partioning swap space
1. mkpart
	a. Filesystem has to be *linux-swap*
2. udevadm settle
3. mkswap
4. swapon or /etc/fstab 
	a. mount point: swap
	b. filesystem type: swap
5. systemctl daemon-reload
6. swapon --show to display swap spaces

### Review from RHCSA - Logical Volumes
#### Creating Logical Volumes
1. Prepare teh physical device
	a. mkpart - set lvm on
2. Create a physical volume
	a. *pvcreate* /dev/vdb1
3. Create a Volume Group
	a. *vgcreate* vgname /dev/vdb1 /dev/vdb2
4. Create a logical volume
	a. *lvcreate* -n lvname -L LVsizeinBytes vgname
5. Add the filesystem
	a. *mkfs*
6. Mount in /etc/fstab (1 2) for options
7. Other useful commands: *lvremove*, *vgremove*, *pvremove*, *lvdisplay*, *vgdisplay*, *pvdisplay*

### Review from RHCSA - Managing Networking
#### Validating Network Configurations
* *ip link show*
#### Configuring Networking from the Command Line
#### Editing Network Configuraiton Files
#### Configuring Host Names and Name Resolution
