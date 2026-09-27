# Troubleshooting

This document records issues encountered while building and configuring the Debian Linux server, along with the investigation, root cause, resolution, and verification steps.

---

## Table of Contents

- [Troubleshooting](#troubleshooting)
  - [Table of Contents](#table-of-contents)
- [SSH Connection Timeout During Initial Setup](#ssh-connection-timeout-during-initial-setup)
  - [Problem](#problem)
  - [Initial Environment](#initial-environment)
    - [Host](#host)
    - [Hypervisor](#hypervisor)
    - [Virtual Network](#virtual-network)
    - [Debian Server](#debian-server)
- [Investigation](#investigation)
  - [Finding 1: SSH Service](#finding-1-ssh-service)
    - [Conclusion](#conclusion)
  - [Finding 2: Debian Network Configuration](#finding-2-debian-network-configuration)
    - [Connectivity Test](#connectivity-test)
    - [Conclusion](#conclusion-1)
  - [Finding 3: Windows Network Configuration](#finding-3-windows-network-configuration)
    - [Interpretation](#interpretation)
  - [Finding 4: VMware Virtual Network](#finding-4-vmware-virtual-network)
    - [Conclusion](#conclusion-2)
- [Root Cause](#root-cause)
- [Resolution](#resolution)
- [Verification](#verification)
  - [ICMP Connectivity Test](#icmp-connectivity-test)
  - [TCP Connectivity Test](#tcp-connectivity-test)
- [Final Network Topology](#final-network-topology)
- [Lessons Learned](#lessons-learned)
  - [1. Verify the service before changing its configuration](#1-verify-the-service-before-changing-its-configuration)
  - [2. Troubleshoot networking layer by layer](#2-troubleshoot-networking-layer-by-layer)
  - [3. An active network adapter does not necessarily mean it has correct IP configuration](#3-an-active-network-adapter-does-not-necessarily-mean-it-has-correct-ip-configuration)
  - [4. APIPA addresses are useful diagnostic information](#4-apipa-addresses-are-useful-diagnostic-information)
  - [5. Routing is critical for connectivity](#5-routing-is-critical-for-connectivity)
  - [6. Connectivity should be verified at multiple layers](#6-connectivity-should-be-verified-at-multiple-layers)
  - [Status](#status)

---

# SSH Connection Timeout During Initial Setup

## Problem

After installing Debian Linux in VMware Workstation Pro and starting the OpenSSH server, an SSH connection from the Windows host to the Debian virtual machine failed with a connection timeout.

The attempted connection was:

```bash
ssh alan@192.168.71.136
````

The connection returned:

```text
ssh: connect to host 192.168.71.136 port 22: Connection timed out
```

The initial assumption was that the SSH service or the Debian firewall might be preventing the connection.

---

## Initial Environment

### Host

* Operating System: Windows
* Physical network: Wi-Fi
* Host IP address: `192.168.1.100`

### Hypervisor

* VMware Workstation Pro

### Virtual Network

* VMware VMnet8
* Network type: NAT
* VMware NAT subnet: `192.168.71.0/24`
* DHCP: Enabled

### Debian Server

* Network interface: `ens33`
* IP address: `192.168.71.136/24`
* Default gateway: `192.168.71.2`
* SSH port: `22`

---

# Investigation

The issue was investigated layer by layer instead of immediately changing the SSH configuration.

The investigation followed this general path:

```text
SSH connection
      ↓
SSH service
      ↓
Server IP configuration
      ↓
Server routing
      ↓
VMware NAT
      ↓
Windows VMnet8 adapter
      ↓
Host-to-VM connectivity
```

---

## Finding 1: SSH Service

The SSH service was checked on the Debian server:

```bash
systemctl status ssh
```

The service was found to be active:

```text
Active: active (running)
```

The service logs also showed that OpenSSH was listening on port 22:

```text
Server listening on 0.0.0.0 port 22.
Server listening on :: port 22.
```

This established that the SSH daemon was running correctly and listening for connections on both IPv4 and IPv6.

### Conclusion

The SSH daemon itself was not the cause of the timeout.

---

## Finding 2: Debian Network Configuration

The Debian server's network configuration was inspected using:

```bash
ip addr
```

The server had the following IPv4 address:

```text
192.168.71.136/24
```

on interface:

```text
ens33
```

The routing table was then checked:

```bash
ip route
```

The result showed:

```text
default via 192.168.71.2 dev ens33 proto dhcp
192.168.71.0/24 dev ens33 proto dhcp scope link src 192.168.71.136
```

This indicated that Debian had:

* A valid IP address
* A connected route to the `192.168.71.0/24` network
* A default route through `192.168.71.2`

### Connectivity Test

The VMware NAT gateway was tested:

```bash
ping -c 4 192.168.71.2
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

The physical network gateway was also tested:

```bash
ping -c 4 192.168.1.1
```

Result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

### Conclusion

The Debian server's network interface, routing table, VMware NAT gateway connectivity, and connectivity toward the physical network were functioning correctly.

The problem was therefore unlikely to be inside the Debian network configuration.

---

## Finding 3: Windows Network Configuration

The Windows network configuration was inspected using:

```powershell
ipconfig
```

The physical Wi-Fi adapter had:

```text
IPv4 Address: 192.168.1.100
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
```

However, the VMware VMnet8 adapter had:

```text
IPv4 Address: 169.254.162.39
Subnet Mask: 255.255.0.0
```

The VMnet8 adapter therefore did not have an address belonging to the VMware NAT subnet:

```text
192.168.71.0/24
```

The Windows host also had no route for:

```text
192.168.71.0/24
```

This was verified with:

```powershell
Get-NetRoute -DestinationPrefix "192.168.71.0/24"
```

which returned no matching route.

The VMnet8 address was inspected directly:

```powershell
Get-NetIPAddress -InterfaceAlias "VMware Network Adapter VMnet8" -AddressFamily IPv4
```

The result was:

```text
IPAddress      : 169.254.162.39
PrefixLength   : 16
```

### Interpretation

The `169.254.0.0/16` address range is an APIPA/link-local address range. The Windows VMnet8 adapter had therefore not been configured with the expected address for the VMware NAT network.

---

## Finding 4: VMware Virtual Network

VMware's Virtual Network Editor was inspected.

The relevant configuration was:

```text
VMnet8
Type: NAT
DHCP: Enabled
Subnet: 192.168.71.0/24
```

The VMware NAT and DHCP services on Windows were also checked:

```powershell
Get-Service "VMware NAT Service"
```

Result:

```text
Status: Running
```

The DHCP service was checked with:

```powershell
Get-Service "VMware DHCP Service"
```

Result:

```text
Status: Running
```

The VMware network adapters themselves were also checked:

```powershell
Get-NetAdapter | Where-Object {$_.Name -like "*VMware*"} | Format-Table Name, Status, LinkSpeed, MacAddress
```

VMnet8 was reported as:

```text
Name: VMware Network Adapter VMnet8
Status: Up
```

### Conclusion

The VMware NAT and DHCP services were running, VMnet8 was configured as a NAT network using `192.168.71.0/24`, and the Debian VM was correctly connected to that network.

The remaining problem was the IP configuration of the Windows-side VMnet8 adapter.

---

# Root Cause

The SSH timeout was caused by an incorrect IP configuration on the Windows host's VMware VMnet8 virtual adapter.

The VMware NAT network was configured as:

```text
192.168.71.0/24
```

and the Debian server was correctly configured as:

```text
192.168.71.136/24
```

However, the Windows VMnet8 adapter had an APIPA address:

```text
169.254.162.39/16
```

instead of an address on the VMware NAT subnet.

As a result, Windows had no route to:

```text
192.168.71.0/24
```

and attempted connections to:

```text
192.168.71.136:22
```

could not reach the Debian server.

The SSH daemon itself was functioning correctly.

---

# Resolution

The incorrect APIPA address was removed from the Windows VMnet8 adapter.

The affected interface was:

```text
Interface Index: 23
Interface: VMware Network Adapter VMnet8
```

The incorrect address was removed using:

```powershell
Remove-NetIPAddress -InterfaceIndex 23 -IPAddress 169.254.162.39 -Confirm:$false
```

The VMnet8 adapter was then assigned an address belonging to the VMware NAT subnet:

```powershell
New-NetIPAddress -InterfaceIndex 23 -IPAddress 192.168.71.1 -PrefixLength 24
```

No default gateway was assigned to the Windows VMnet8 adapter.

The resulting configuration was:

```text
Windows VMnet8:
192.168.71.1/24
```

This placed the Windows host and Debian server on the same VMware virtual subnet.

---

# Verification

The Windows VMnet8 adapter was checked again:

```powershell
Get-NetIPAddress -InterfaceAlias "VMware Network Adapter VMnet8" -AddressFamily IPv4
```

The resulting configuration was:

```text
IPAddress      : 192.168.71.1
PrefixLength   : 24
AddressState   : Preferred
```

The corresponding route was then present:

```powershell
Get-NetRoute -DestinationPrefix "192.168.71.0/24"
```

Result:

```text
DestinationPrefix : 192.168.71.0/24
InterfaceIndex    : 23
NextHop           : 0.0.0.0
```

## ICMP Connectivity Test

The Debian server was tested from Windows:

```powershell
ping 192.168.71.136
```

Result:

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

This confirmed successful network connectivity between the Windows host and Debian server.

## TCP Connectivity Test

The SSH port was then tested:

```powershell
Test-NetConnection 192.168.71.136 -Port 22
```

Result:

```text
ComputerName     : 192.168.71.136
RemoteAddress    : 192.168.71.136
RemotePort       : 22
InterfaceAlias   : VMware Network Adapter VMnet8
SourceAddress    : 192.168.71.1
TcpTestSucceeded : True
```

This confirmed that TCP port 22 was reachable from the Windows host.

SSH access could therefore be established using:

```powershell
ssh alan@192.168.71.136
```

---

# Final Network Topology

After resolving the issue, the relevant network topology became:

```text
                         Physical Network
                         192.168.1.0/24
                                │
                                │
                         Home Router
                         192.168.1.1
                                │
                                │
                       Windows Host
                       192.168.1.100
                                │
                                │
                       VMware Workstation
                                │
                         ┌──────┴──────┐
                         │   VMnet8    │
                         │     NAT     │
                         │192.168.71.0/24
                         └──────┬──────┘
                                │
               ┌────────────────┼────────────────┐
               │                │                │
               │                │                │
       Windows VMnet8     VMware NAT        Debian Server
       192.168.71.1       192.168.71.2      192.168.71.136
                                                │
                                                │
                                           SSH : TCP/22
                                                │
                                                ▼
                                         Remote Administration
```

---

# Lessons Learned

## 1. Verify the service before changing its configuration

The SSH daemon was already running and listening on port 22.

Checking:

```bash
systemctl status ssh
```

prevented unnecessary changes to the SSH configuration.

## 2. Troubleshoot networking layer by layer

The issue was isolated by checking:

```text
SSH service
    ↓
Server IP configuration
    ↓
Server routing
    ↓
NAT gateway connectivity
    ↓
VMware network configuration
    ↓
Windows virtual adapter
    ↓
Host-to-server connectivity
```

This prevented unrelated configuration changes from being introduced.

## 3. An active network adapter does not necessarily mean it has correct IP configuration

The VMware VMnet8 adapter reported:

```text
Status: Up
```

but its IP configuration was incorrect.

The adapter therefore existed and was operational at the link level, but it was not correctly configured for the VMware NAT subnet.

## 4. APIPA addresses are useful diagnostic information

The address:

```text
169.254.162.39
```

indicated that the Windows VMnet8 adapter did not have the expected address configuration.

## 5. Routing is critical for connectivity

Windows initially had no route to:

```text
192.168.71.0/24
```

After configuring the VMnet8 adapter as:

```text
192.168.71.1/24
```

Windows automatically gained the connected route to the subnet.

## 6. Connectivity should be verified at multiple layers

The issue was considered resolved only after verifying:

1. ICMP connectivity:

```powershell
ping 192.168.71.136
```

2. TCP connectivity:

```powershell
Test-NetConnection 192.168.71.136 -Port 22
```

3. Application-level connectivity:

```powershell
ssh alan@192.168.71.136
```

This provides stronger evidence than relying on a single connectivity test.

---

## Status

**Resolved**

The Windows host can now reach the Debian server over VMware's NAT network, and SSH is available for remote administration.