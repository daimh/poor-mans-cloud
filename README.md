# Poor Man's Cloud (PMC)

Build and manage a simple, high-availability cloud with open-source software and commodity hardware. Poor Man's Cloud (PMC) is minimalistic: it requires only one configuration file on the manager node, and no daemons run on the physical hosts.

## Table of Contents

- [Features](#features)
- [Installation](#installation)
- [Setting Up a VM](#setting-up-a-vm)
- [Booting PMs and VMs](#booting-pms-and-vms)
- [Notes](#notes)
- [Status](#status)
- [License](#license)
- [Acknowledgments](#acknowledgments)

## Features

- **100% redundancy:** Every disk write is instantly copied from the active node to its hot standby node (thanks to DRBD).  
- **Live migration:** VMs can be moved within the cloud of nodes without interruption.  
- **Two levels of disk image backup:**  
  - Full backup: `dd` image  
  - Incremental backup: `xdelta3` copy  
- **VM cloning**

## Installation

<details>
<summary>Click to expand installation steps</summary>

Assuming you have two nodes, `pm-1` and `pm-2`, with `pm-1` as the manager node:

1. On the manager node, set up root SSH key access to `pm-2`.

2. On the manager node, run:
   ```bash
   cd /opt
   git clone https://github.com/daimh/poor-mans-cloud.git
   ln -s /opt/poor-mans-cloud/etc/pmc.conf /etc/pmc.conf
   export PATH=$PATH:/opt/poor-mans-cloud/bin
   pmc-pm-init pm-1
   pmc-pm-init pm-2
   ```

`pmc-pm-init` checks DRBD, Python, libvirtd, a bridge network interface `br0`, the volume group defined in `/etc/pmc.conf`, and the VM backup directory. Verify with your network administrator before enabling the bridge interface.

</details>

## Setting Up a VM

<details>
<summary>Click to expand installation steps</summary>

1. Copy your installation ISO to `/opt/poor-mans-cloud/usr/lib/pmc/iso/`. Example: `ubuntu-2020.iso`.

2. Add a VM:

   ```bash
   pmc-vm-add -p pm-1 -p pm-2 -s 50 -m 4 -i ubuntu-2020 -a N.N.N.N test-vm
   ```

   * `-p pm-1 -p pm-2`: VM runs on pm-1; pm-2 holds the DRBD copy of the VM's storage
   * `-s 50`: 50 GB storage
   * `-m 4`: 4 GB RAM
   * `-i ubuntu-2020`: ISO used during installation
   * `-a N.N.N.N`: IP address if DNS cannot resolve `test-vm`
   * `test-vm`: VM name

3. After installation, check status:

   ```bash
   pmc-vm-ls
   pmc-pm-ls
   ```

4. Live migrate a VM:

   ```bash
   pmc-vm-mv test-vm
   ```

5. Back up the VM:

   ```bash
   pmc-vm-backup -b test-vm   # Full dd image
   pmc-vm-backup test-vm       # Incremental xdelta3 copy
   ```

6. Remove a VM:

   ```bash
   pmc-vm-rm test-vm
   ```

</details>

## Booting PMs and VMs

<details>
<summary>Click to expand installation steps</summary>

1. Power on all physical hosts.

2. Start DRBD:

   ```bash
   pmc-pm-drbd-up pm-1
   pmc-pm-drbd-up pm-2
   ```

3. Check DRBD status:

   ```bash
   ssh pm-1 drbdadm status
   ssh pm-2 drbdadm status
   ```

4. Start the VM on the desired node:

   ```bash
   ssh pm-1 virsh start test-vm
   ```

</details>

## Notes

<details>
<summary>Click to expand installation steps</summary>

PMC is designed for small setups. It **does not store VM information centrally**; each command queries physical hosts for real-time status. In tests (~20 VMs on 5 physical hosts), performance is good, but it may not scale to hundreds of hosts or thousands of VMs.

</details>

## Status

<details>
<summary>Click to expand installation steps</summary>

| Date       | Libvirt     | Linux           | DRBD Utils     |
| ---------- | ----------- | --------------- | -------------- |
| 2025-12-06 | 1:11.10.0-1 | 6.17.9.arch1-1  | 9.21.1-2 (AUR) |
| 2025-02-16 | 1:11.0.0-2  | 6.12.10.arch1-1 | 9.21.1-2 (AUR) |
| 2021-09-29 | 1:7.7.0-1   | 5.14.8.arch1-1  | 9.16.0-1 (AUR) |
| 2021-02-06 | 1:7.0.0-2   | 5.10.13.arch1-1 | 9.16.0-1 (AUR) |
| 2020-11-12 | 1:6.5.0-3   | 5.9.8.arch1-1   | 9.11.0-1 (AUR) |
| 2019-12    | 5.8.0-2     | 5.3.10.1-1      | 9.11.0-1 (AUR) |

</details>

## License

<details>
<summary>Click to expand installation steps</summary>

Developed by [Manhong Dai](mailto:manhongdai@gmail.com)

Copyright © 2020-2025 University of Michigan. MIT License.

This is free software: you are free to use, modify, and redistribute it under the terms of the MIT License. There is **NO WARRANTY**, to the extent permitted by law.

</details>

## Acknowledgments

<details>
<summary>Click to expand installation steps</summary>

* Andy Lin, MNI, UMICH
* Julie Gales, Administrator, MNI, UMICH
* Ruth Freedman, MPH, Former Administrator, MNI, UMICH
* Fan Meng, Ph.D., Research Associate Professor, Psychiatry, UMICH
* Huda Akil, Ph.D., Director, MNI, UMICH
* Stanley J. Watson, M.D., Ph.D., Director, MNI, UMICH

</details>
