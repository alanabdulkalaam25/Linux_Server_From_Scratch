# Phase 1 — Server Installation

## Objective

The objective of this phase is to install a minimal Debian Linux server inside a VMware Workstation Pro virtual machine.

The installation provides the clean baseline environment on which the remaining Linux server administration tasks will be performed.

---

## Environment

| Component             | Configuration                   |
| --------------------- | ------------------------------- |
| Hypervisor            | VMware Workstation Pro          |
| Operating System      | Debian GNU/Linux 13 (trixie)    |
| Installation Type     | Virtual Machine                 |
| Architecture          | x86-64 / amd64                  |
| Host Operating System | Windows                         |
| Firmware              | UEFI                            |
| VM CPU                | 2 vCPU                          |
| VM Memory             | 4 GB                            |
| Virtual Disk          | 25 GB                           |
| Network Mode          | VMware NAT                      |
| Server Hostname       | `DEB-SERVER`                    |
| Purpose               | Linux server administration lab |

---

## Installation Plan

The server is being built incrementally from a clean Debian installation.

The initial installation includes:

- A dedicated VMware virtual machine
- 2 virtual CPUs
- 4 GB virtual memory
- 25 GB virtual disk
- UEFI firmware
- VMware NAT networking
- Debian GNU/Linux 13
- A non-root administrative user
- A dedicated server hostname
- Minimal server-oriented software
- OpenSSH server for remote administration

Additional services and security configuration will be introduced in later phases.

---

# Installation

## Debian ISO

The Debian installation ISO was downloaded from the official Debian project website and used as the installation media for the virtual machine.

The installed operating system is:

```text
Debian GNU/Linux 13 (trixie)
````

---

## Virtual Machine Configuration

The Debian server was created as a virtual machine in VMware Workstation Pro with the following hardware configuration:

| Resource | Configuration |
| -------- | ------------- |
| CPU      | 2 vCPU        |
| RAM      | 4 GB          |
| Storage  | 25 GB         |
| Network  | NAT           |
| Firmware | UEFI          |

The virtual disk was partitioned using the Debian installer.

The resulting storage layout is:

| Device      |     Size | Purpose               |
| ----------- | -------: | --------------------- |
| `/dev/sda1` |  ~976 MB | EFI System Partition  |
| `/dev/sda2` | ~22.7 GB | Root filesystem (`/`) |
| `/dev/sda3` |  ~1.3 GB | Swap                  |

---

## Debian Installation

Debian GNU/Linux 13 was successfully installed on the virtual disk.

The installation was performed without a desktop environment because the purpose of the virtual machine is to function as a server rather than a desktop workstation.

A non-root administrative user was created for normal system administration.

The server hostname was configured as:

```text
DEB-SERVER
```

The OpenSSH server was also installed to allow remote administration from the Windows host.

SSH configuration and security hardening will be covered in a later phase.

---

# Initial Network Configuration

The Debian server is connected to VMware's NAT network.

The primary network interface is:

```text
ens33
```

The server received the following IPv4 configuration:

```text
IPv4 Address : 192.168.71.136/24
Gateway      : 192.168.71.2
Network      : 192.168.71.0/24
```

The VMware NAT network is:

```text
192.168.71.0/24
```

The Debian server received its address through DHCP.

At this stage, the server is reachable from the Windows host through VMware's virtual network.

---

# Verification

The following commands were used to establish the initial state of the newly installed server:

```bash
cat /etc/os-release
uname -a
hostnamectl
ip addr
ip route
lsblk
free -h
df -h
```

---

## Operating System

Command:

```bash
cat /etc/os-release
```

Output:

```text
PRETTY_NAME="Debian GNU/Linux 13 (trixie)"
NAME="Debian GNU/Linux"
VERSION_ID="13"
VERSION="13 (trixie)"
VERSION_CODENAME=trixie
DEBIAN_VERSION_FULL=13.7
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```

This confirms that Debian GNU/Linux 13 (trixie) is installed.

---

## Kernel

Command:

```bash
uname -a
```

Output:

```text
Linux DEB-SERVER 6.12.107+deb13-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.12.107-1 (2026-08-29) x86_64 GNU/Linux
```

The system is running the Debian 13 x86-64 kernel:

```text
6.12.107+deb13-amd64
```

---

## Hostname and Virtualization

Command:

```bash
hostnamectl
```

Relevant output:

```text
Static hostname: DEB-SERVER
       Icon name: computer-vm
      Chassis: vm
  Virtualization: vmware
Operating System: Debian GNU/Linux 13 (trixie)
          Kernel: Linux 6.12.107+deb13-amd64
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware20,1
Firmware Version: VMW201.00V.24866131.B64.2507211911
```

The `hostnamectl` command also reported:

```text
Failed to query product UUID, ignoring: Access denied
Failed to query hardware serial, ignoring: Access denied
```

These messages do not indicate an operating system installation failure. They occur because the VMware virtual machine does not expose certain physical hardware metadata to the guest system.

The important system information was still successfully retrieved.

---

## Network Interfaces

Command:

```bash
ip addr
```

Relevant output:

```text
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:0c:29:a7:38:c9 brd ff:ff:ff:ff:ff:ff
    altname enp2s1
    altname enx000c29a738c9
    inet 192.168.71.136/24 brd 192.168.71.255 scope global dynamic noprefixroute ens33
       valid_lft 1176sec preferred_lft 951sec
```

The primary network interface is:

```text
ens33
```

with IPv4 address:

```text
192.168.71.136/24
```

The loopback interface is also available at:

```text
127.0.0.1/8
```

---

## Routing

Command:

```bash
ip route
```

Output:

```text
default via 192.168.71.2 dev ens33 proto dhcp src 192.168.71.136 metric 1002
192.168.71.0/24 dev ens33 proto dhcp scope link src 192.168.71.136 metric 1002
```

The routing table confirms:

* The local network is `192.168.71.0/24`
* The server's address is `192.168.71.136`
* The VMware NAT gateway is `192.168.71.2`
* A default route is available through the VMware NAT gateway

---

## Storage

Command:

```bash
lsblk
```

Output:

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   25G  0 disk
├─sda1   8:1    0  976M  0 part /boot/efi
├─sda2   8:2    0 22.7G  0 part /
└─sda3   8:3    0  1.3G  0 part [SWAP]
sr0     11:0    1  756M  0 rom
```

The virtual disk is approximately 25 GB.

The partition layout consists of:

```text
/dev/sda1 → EFI System Partition
/dev/sda2 → Root filesystem
/dev/sda3 → Swap
```

---

## Memory

Command:

```bash
free -h
```

Output:

```text
               total       used        free       shared    buff/cache   available
Mem:           3.8Gi       397Mi       3.4Gi       744Ki       187Mi       3.4Gi
Swap:          1.3Gi          0B       1.3Gi
```

The server has approximately 4 GB of allocated memory, of which approximately 3.8 GiB is available to the guest operating system.

The system also has approximately 1.3 GiB of configured swap space.

---

## Filesystem Usage

Command:

```bash
df -h
```

The primary root filesystem is:

```text
/dev/sda2
```

with approximately:

```text
23 GB total
1.1 GB used
21 GB available
5% usage
```

The EFI system partition is:

```text
/dev/sda1
```

with approximately:

```text
975 MB total
9.1 MB used
966 MB available
```

The remaining entries reported by `df` are temporary, virtual, runtime, or system-managed filesystems.

---

# Baseline Summary

The initial state of the Debian server is:

| Category          | Value                        |
| ----------------- | ---------------------------- |
| Operating System  | Debian GNU/Linux 13 (trixie) |
| Kernel            | 6.12.107+deb13-amd64         |
| Architecture      | x86-64                       |
| Hypervisor        | VMware Workstation Pro       |
| Hostname          | `DEB-SERVER`                 |
| CPU               | 2 vCPU                       |
| Memory            | 4 GB                         |
| Storage           | 25 GB                        |
| Root Filesystem   | `/dev/sda2`                  |
| Swap              | `/dev/sda3`                  |
| Network Interface | `ens33`                      |
| IPv4 Address      | `192.168.71.136/24`          |
| Default Gateway   | `192.168.71.2`               |
| Network Mode      | VMware NAT                   |
| SSH               | Installed and running        |

---

# Result

The Debian server was successfully installed inside VMware Workstation Pro.

The system boots successfully, the configured non-root administrative user can log in, and the initial operating system, hardware, storage, memory, and network configuration have been verified.

The server is now accessible remotely through SSH from the Windows host.

The resulting environment provides the baseline for the subsequent Linux server administration phases.

---

# Issues and Troubleshooting

During the initial setup, an SSH connectivity problem was encountered.

The Debian SSH service was running correctly, but the Windows host could not initially reach the Debian server because the VMware VMnet8 adapter on Windows had an incorrect APIPA address (`169.254.162.39/16`) instead of an address on the VMware NAT subnet (`192.168.71.0/24`).

The issue was resolved by configuring the Windows VMnet8 adapter with:

```text
192.168.71.1/24
```

Connectivity was then verified using:

```powershell
ping 192.168.71.136
```

and:

```powershell
Test-NetConnection 192.168.71.136 -Port 22
```

Both tests succeeded.

Detailed investigation and resolution are documented in:

[`troubleshooting.md`](troubleshooting.md)

---

# Phase Status

**Status: Completed**

Phase 1 established a clean, verified Debian Linux server environment that will be used as the foundation for the remaining server administration work.
