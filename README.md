# poor-mans-cloud
build and manage a simple but high-availability cloud with open source software and commondity hardware. PMC has the simplest configuration, as it requires only one configuration file on its manager node, and there is no running daemons on any nodes.

## Features
- 100% redundancy. Every disk write is instantly copied from an active node to a hot standby node.

- live migration. VM can be moved within a cloud of nodes without any user interruption.

- two levels of disk image backup, full backup is a dd image, incremental backup is based on xdelta3.

- clone

## Installation
1. Assuming we have two nodes, pm-1 and pm-2. We also need a manager node that can run X application and also ssh to all the nodes

2. go to the manager node, setup a ssh key access to all nodes

3. on the manager node  
  $ cd /opt  
  $ git clone https://github.com/daimh/poor-mans-cloud.git  
  $ ln -s /opt/poor-mans-cloud/etc/pmc.conf /etc/pmc.conf  
  $ export PATH=$PATH:/opt/poor-mans-cloud/bin  
  $ pmc-pm-init pm-1  
  $ pmc-pm-init pm-2  

  pmc-pm-init will check drbd, pyton, libvirtd, a bridge network interface 'br0', a volume group defined in /etc/pmc.conf, and the VM backup folder. Please check with your network administrator before enabling bridge device.

## Setup a VM
1. go to the manager node

2. copy an installation iso file to /opt/poor-mans-cloud/usr/lib/pmc/iso/. Following commands assume it is 'ubuntu-2020.iso'

3. $ pmc-vm-add -p pm-1 -p pm-2 -s 50 -m 4 -i ubuntu-2020 -a N.N.N.N test-vm  
  -p pm-1 -p pm-2, the VM will run on pm-1, pm-2 has a drbd copy of the VM's storage  
  -s 50, 50G storage  
  -m 4, 4G ram  
  -i ubuntu-2020, the iso file will be used as CD-DVD during installation  
  -a N.N.N.N, ip address, if your DNS server cannot resolve 'test-vm'  
  test-vm, VM name

4. after the VM's OS installation, you can run 'pmc-vm-ls' or 'pmc-pm-ls' to list its status.

5. Live migrate the VM  
  $ pmc-vm-mv test-vm 

5. back it up  
  $ pmc-vm-backup -b test-vm #dd image  
  $ pmc-vm-backup test-vm #incremental xdelta3 copy  

6. remove the VM  
  $ pmc-vm-rm test-vm 

## Boot up all nodes and the VM
1. start all nodes

2. start drbd  
  $ pmc-pm-ready-drbd pm-1  
  $ pmc-pm-ready-drbd pm-2  

3. check all drbd status  
  $ ssh pm-1 cat /proc/drbd  
  $ ssh pm-2 cat /proc/drbd  

4. go to the node you want to start 'test-vm' and run  
  $ virsh start test-vm

## Comments
  In short, PMC is for poor man only. If you have a million dollar, go to VMware, or we can work together to develop a Rich Man's Cloud.

  PMC doesn't store the VM information anywhere. Every command requires the manager node to go through each physical node to get the real time status. In my case of ~20 VM running on 5 physical nodes, PMC performance is pretty good. But I guess it won't scale well if there are thousands of VM running on hundreds of physical nodes.

## Status
-  2021-02-06, libvirt-1:7.0.0-2, linux-5.10.13.arch1-1 and drbd-utils-9.16.0-1 (AUR)
-  2020-11-12, libvirt-1:6.5.0-3, linux-5.9.8.arch1-1 and drbd-utils-9.11.0-1 (AUR)
-  2019-12, PMC was running stably with libvirt-5.8.0-2, linux-5.3.10.1-1 and drbd-utils-9.11.0-1 (AUR) on Arch Linux. VM consists of Arch Linux, and a few Windows versions.  

## Copyright

Developed by [Manhong Dai](mailto:daimh@umich.edu)

Copyright © 2020 University of Michigan. License [GPLv3+](https://gnu.org/licenses/gpl.html): GNU GPL version 3 or later 

This is free software: you are free to change and redistribute it.

There is NO WARRANTY, to the extent permitted by law.

## Acknowledgment

Ruth Freedman, MPH, former administrator of MNI, UMICH

Fan Meng, Ph.D., Research Associate Professor, Psychiatry, UMICH

Huda Akil, Ph.D., Director of MNI, UMICH

Stanley J. Watson, M.D., Ph.D., Director of MNI, UMICH
