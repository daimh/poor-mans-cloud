# poor-mans-cloud
build and manage a cloud with libvirtd, drbd and commodity hardware 

1, PMC (Poor Man's Cloud) has 100% redundancy. Every disk IO operation is instantly synchronized from the running PM (physical machine) to the standby one. Asssuming the worst-case scenario, a physical machine running a VM (Virtual Machine) is destroyed accidently, admin can boot up the same VM on the standby PM immediately. Then admin can choose a new PM to clone the rescued VM with one command, and the VM will have redundancy again.

2, PMC supports live migration. VM can be migrated from its running PM to the standby one without any downtime.

3, PMC provides two level of backup, full backup is a dd image, incremental backup is based on xdelta3.
