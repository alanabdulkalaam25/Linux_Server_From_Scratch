# Linux Server From Scratch

A hands-on Linux server administration project built from a fresh Debian Linux installation.

The purpose of this project is to build, configure, secure, automate, and document a Linux server from the ground up while developing practical Linux system administration skills.

This project is being developed incrementally, with each phase focusing on a specific area of Linux server administration and documenting the configuration, commands, verification process, troubleshooting, and lessons learned.

---

## Project Objectives

The project aims to provide practical experience with:

- Installing and configuring a Linux server
- Understanding Linux system administration
- Managing users, groups, permissions, and privileges
- Configuring networking and remote administration
- Configuring and securing SSH
- Implementing firewall rules
- Deploying and configuring a web server
- Managing services with `systemd`
- Understanding Linux logging and system diagnostics
- Implementing server health checks and monitoring
- Automating administrative tasks with Bash
- Implementing automated backups
- Troubleshooting real-world server and networking problems
- Maintaining professional infrastructure documentation

---

## Technology Stack

| Category              | Technology                   |
| --------------------- | ---------------------------- |
| Operating System      | Debian GNU/Linux 13 (trixie) |
| Hypervisor            | VMware Workstation Pro       |
| Remote Administration | OpenSSH                      |
| Web Server            | Nginx                        |
| Service Management    | systemd                      |
| Firewall              | nftables / firewall tooling  |
| Scripting             | Bash                         |
| Version Control       | Git                          |
| Repository Hosting    | GitHub                       |

Additional tools and technologies may be introduced as the project progresses.

---

## Server Environment

The initial server environment is configured as follows:

| Resource          | Configuration                |
| ----------------- | ---------------------------- |
| Operating System  | Debian GNU/Linux 13 (trixie) |
| Architecture      | x86-64                       |
| Hypervisor        | VMware Workstation Pro       |
| CPU               | 2 vCPU                       |
| Memory            | 4 GB                         |
| Storage           | 25 GB                        |
| Firmware          | UEFI                         |
| Network           | VMware NAT                   |
| Hostname          | `DEB-SERVER`                 |
| Network Interface | `ens33`                      |
| IPv4 Address      | `192.168.71.136/24`          |

The server is deployed as a virtualized laboratory environment on a Windows host.

---

## Project Architecture

The server will be developed progressively from a minimal Debian installation into a more complete server environment.

```text
                    Windows Host
                         │
                         │
                 VMware Workstation
                         │
                    VMware NAT
                  192.168.71.0/24
                         │
                         │
                 ┌───────▼────────┐
                 │   DEB-SERVER   │
                 │ Debian 13      │
                 │ 2 vCPU / 4 GB  │
                 │ 25 GB Storage  │
                 └───────┬────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
             SSH       Nginx     systemd
              │          │          │
              │          │          │
        Administration  Web      Services
````

The architecture will be expanded as additional components are introduced.

---

## Project Phases

### Phase 1 — Server Installation

**Status: Completed**

Installed and verified the initial Debian server environment.

Covered:

* VMware virtual machine creation
* Debian GNU/Linux 13 installation
* UEFI configuration
* CPU, memory, and storage configuration
* Server hostname configuration
* Initial network configuration
* OpenSSH installation
* System verification
* Storage and memory verification
* Initial SSH connectivity
* Troubleshooting VMware NAT networking

Documentation:

* [`Phase 1 — Server Installation`](docs/phase-01-installation.md)
* [`Troubleshooting`](docs/troubleshooting.md)

---

### Phase 2 — Linux System Administration

**Status: Planned**

Topics will include:

* Linux filesystem hierarchy
* Users and groups
* User and group management
* File ownership
* Linux permissions
* `sudo`
* Administrative privileges
* Process management
* Package management
* Environment variables
* Shell configuration

---

### Phase 3 — Networking and Remote Administration

**Status: Planned**

Topics will include:

* Network interfaces
* IP addressing
* Routing
* DNS
* Network diagnostics
* SSH configuration
* SSH key authentication
* SSH security hardening
* Remote administration

---

### Phase 4 — Server Security

**Status: Planned**

Topics will include:

* Firewall configuration
* nftables
* SSH hardening
* Service exposure
* User privilege management
* Security auditing
* Basic attack-surface reduction

---

### Phase 5 — Web Server Deployment

**Status: Planned**

Topics will include:

* Nginx installation
* Nginx configuration
* Virtual hosts / server blocks
* Static website deployment
* Access and error logs
* Service management
* Web server troubleshooting

---

### Phase 6 — systemd and Service Management

**Status: Planned**

Topics will include:

* systemd architecture
* Units
* Services
* Targets
* Service lifecycle
* Creating custom services
* Service dependencies
* Automatic service startup
* Service troubleshooting

---

### Phase 7 — Logging and Monitoring

**Status: Planned**

Topics will include:

* system logs
* `journalctl`
* Log analysis
* Resource monitoring
* CPU and memory usage
* Disk usage
* Process monitoring
* Server health checks
* Basic monitoring automation

---

### Phase 8 — Automation

**Status: Planned**

Topics will include:

* Bash scripting
* Administrative automation
* Scheduled tasks
* `cron` / systemd timers
* Automated health checks
* Maintenance scripts

---

### Phase 9 — Backup and Recovery

**Status: Planned**

Topics will include:

* Backup strategy
* File backups
* Configuration backups
* Automated backups
* Backup verification
* Recovery procedures
* Restore testing

---

### Phase 10 — Final Server Hardening and Documentation

**Status: Planned**

The final phase will consolidate the server configuration and document:

* Final architecture
* Security configuration
* Services
* Network configuration
* Automation
* Backup strategy
* Monitoring
* Troubleshooting procedures
* Operational procedures
* Lessons learned

---

## Documentation

Detailed documentation is maintained in the [`docs/`](docs/) directory.

Each phase documents:

1. Objective
2. Environment
3. Configuration
4. Commands used
5. Explanation of the configuration
6. Verification
7. Troubleshooting
8. Result
9. Lessons learned

This approach is intended to make the project reproducible rather than simply recording a list of commands.

---

## Repository Structure

```text
linux-server-from-scratch/
│
├── README.md
│
├── docs/
│   ├── phase-01-installation.md
│   └── troubleshooting.md
│
├── configs/
│
├── scripts/
│
└── screenshots/
```

Additional files and directories will be added as the project develops.

---

## Troubleshooting

Real configuration problems encountered during the project are documented rather than omitted.

For example, during Phase 1, SSH was initially unreachable from the Windows host even though the SSH service was running correctly on Debian.

The investigation identified an incorrect IP configuration on the Windows VMware VMnet8 adapter, which prevented communication with the VMware NAT subnet.

The complete investigation and resolution are documented in:

[`docs/troubleshooting.md`](docs/troubleshooting.md)

---

## Project Status

**Current Phase: Phase 1 — Server Installation**

**Status: Completed**

The Debian server has been successfully installed, configured, and verified inside VMware Workstation Pro.

The project will continue incrementally, with each subsequent phase adding another layer of administration, security, deployment, automation, and operational capability.

---

## Learning Objectives

By completing this project, the goal is to develop practical understanding of:

* Linux system administration
* Server networking
* Remote system administration
* Linux security
* Service management
* Web server administration
* Shell scripting
* Troubleshooting
* Monitoring
* Backup and recovery
* Infrastructure documentation
* Git-based infrastructure workflows

---

## Disclaimer

This project is a personal Linux administration laboratory.

The server is running inside a virtualized environment and is intended for learning, experimentation, configuration practice, and documentation.

```

### One important change

I deliberately moved the detailed **Phase 1 verification output** out of the main README. Your README should answer:

> **What is this project, what has been built, how is it structured, and where can I find the technical details?**

Your `docs/phase-01-installation.md` should answer:

> **Exactly how was the server installed and verified?**

And `docs/troubleshooting.md` should answer:

> **What went wrong, how was it diagnosed, and how was it fixed?**

That separation makes the repository look much more like an actual infrastructure project rather than a collection of terminal notes.
```
