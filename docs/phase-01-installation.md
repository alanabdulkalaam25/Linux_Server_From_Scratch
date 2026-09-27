# Phase 1 — Server Installation

## Objective

The objective of this phase is to install a minimal Debian Linux server inside a VMware Workstation Pro virtual machine.

The installation will provide the clean baseline environment on which the remaining Linux server administration tasks will be performed.

## Environment

| Component             | Configuration                   |
| --------------------- | ------------------------------- |
| Hypervisor            | VMware Workstation Pro          |
| Operating System      | Debian Linux                    |
| Installation Type     | Virtual Machine                 |
| Architecture          | x86_64 / amd64                  |
| Host Operating System | Windows                         |
| Purpose               | Linux server administration lab |

## Installation Plan

The server will be configured with:

* A dedicated virtual machine
* Virtual CPU resources
* Dedicated virtual memory
* Virtual disk storage
* Network connectivity
* Debian Linux installation
* A non-root administrative user
* Appropriate hostname
* Minimal server-oriented software

## Installation

The Debian ISO was downloaded from the official Debian project website and will be used as the installation media for the virtual machine.

### Virtual Machine Configuration

This section will document the hardware configuration selected for the Debian server.

| Resource | Configuration |
| -------- | ------------- |
| CPU      | TBD           |
| RAM      | TBD           |
| Storage  | TBD           |
| Network  | TBD           |
| Firmware | TBD           |

### Debian Installation

The Debian installer will be used to install the operating system onto the virtual disk.

The exact installation choices and configuration will be documented after completing the installation.

## Verification

The following information will be collected after installation:

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

These commands will establish the initial state of the newly installed server.

## Result

*To be completed after installation.*

## Issues and Troubleshooting

*No issues recorded yet.*
