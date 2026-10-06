# Phase 2 Troubleshooting

This document records troubleshooting and unexpected behavior encountered during **Phase 2 — Linux System Administration** of the Debian Linux server project.

The issues documented here occurred while working with the Linux filesystem, users and groups, ownership, permissions, sudo, process management, and package management.

---

## Table of Contents

- [Phase 2 Troubleshooting](#phase-2-troubleshooting)
  - [Table of Contents](#table-of-contents)
- [Filesystem Inspection](#filesystem-inspection)
  - [Transient `/proc` Errors During Disk Usage Analysis](#transient-proc-errors-during-disk-usage-analysis)
    - [Problem](#problem)
    - [Investigation](#investigation)
    - [Resolution](#resolution)
    - [Conclusion](#conclusion)
    - [Lesson](#lesson)
- [Users, Groups, Ownership and Permissions](#users-groups-ownership-and-permissions)
  - [Access Denied When Accessing `labuser` Home Directory](#access-denied-when-accessing-labuser-home-directory)
    - [Problem](#problem-1)
    - [Investigation](#investigation-1)
    - [Resolution](#resolution-1)
    - [Conclusion](#conclusion-1)
  - [Group Membership Removal and Re-addition](#group-membership-removal-and-re-addition)
    - [Problem](#problem-2)
    - [Verification](#verification)
    - [Resolution](#resolution-2)
    - [Conclusion](#conclusion-2)
  - [Shared Directory Access Testing](#shared-directory-access-testing)
    - [Problem](#problem-3)
    - [Investigation](#investigation-2)
    - [Resolution](#resolution-3)
    - [Conclusion](#conclusion-3)
- [Sudo and Administrative Privileges](#sudo-and-administrative-privileges)
  - [Restricted Access to `/etc/shadow`](#restricted-access-to-etcshadow)
    - [Problem](#problem-4)
    - [Investigation](#investigation-3)
    - [Resolution](#resolution-4)
    - [Conclusion](#conclusion-4)
  - [Missing `/var/log/auth.log`](#missing-varlogauthlog)
    - [Problem](#problem-5)
    - [Investigation](#investigation-4)
    - [Resolution](#resolution-5)
    - [Conclusion](#conclusion-5)
- [Process Management](#process-management)
  - [`pstree` Command Not Available](#pstree-command-not-available)
    - [Problem](#problem-6)
    - [Investigation](#investigation-5)
    - [Resolution](#resolution-6)
    - [Conclusion](#conclusion-6)
- [Package Management](#package-management)
  - [Removed Package Still Resolved by Bash](#removed-package-still-resolved-by-bash)
    - [Problem](#problem-7)
    - [Investigation](#investigation-6)
    - [Resolution](#resolution-7)
    - [Conclusion](#conclusion-7)
    - [Lesson](#lesson-1)
- [Lessons Learned](#lessons-learned)
  - [1. Virtual filesystems require special consideration](#1-virtual-filesystems-require-special-consideration)
  - [2. Directory permissions are as important as file permissions](#2-directory-permissions-are-as-important-as-file-permissions)
  - [3. User, group and permission changes should always be verified](#3-user-group-and-permission-changes-should-always-be-verified)
  - [4. Administrative privileges should be used deliberately](#4-administrative-privileges-should-be-used-deliberately)
  - [5. Systemd journal can provide administrative activity logs](#5-systemd-journal-can-provide-administrative-activity-logs)
  - [6. Minimal server installations may not contain every utility](#6-minimal-server-installations-may-not-contain-every-utility)
  - [7. Shell state can affect command verification](#7-shell-state-can-affect-command-verification)
- [Status](#status)

---

# Filesystem Inspection

## Transient `/proc` Errors During Disk Usage Analysis

### Problem

While inspecting disk usage from the filesystem root, the following command was used:

```bash
sudo du -sh /*
```

The command produced warnings similar to:

```text
du: cannot access '/proc/<PID>/...': No such file or directory
```

### Investigation

The warnings occurred while `du` was traversing the `/proc` filesystem.

`/proc` is a virtual filesystem containing information about currently running processes. Process entries can disappear while `du` is scanning them because processes may terminate during the traversal.

Therefore, the filesystem itself was not corrupted and the warnings did not indicate a disk failure.

### Resolution

Disk usage was inspected again while suppressing the transient warnings and restricting the calculation to the actual filesystem:

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

The resulting disk usage information was successfully obtained.

### Conclusion

The warnings were caused by the dynamic nature of `/proc`, not by a problem with the Debian filesystem.

### Lesson

Virtual filesystems such as `/proc` should be treated differently from persistent filesystems when performing disk usage analysis.

---

# Users, Groups, Ownership and Permissions

## Access Denied When Accessing `labuser` Home Directory

### Problem

During ownership and permission testing, the file:

```text
/home/labuser/ownership-test.txt
```

was created and assigned to:

```text
labuser:labadmins
```

However, the `alan` user could not directly access the file.

An access attempt resulted in a permission denial.

### Investigation

The home directory was checked:

```bash
ls -ld /home/labuser
```

The directory had restrictive permissions:

```text
drwx------ labuser labuser /home/labuser
```

The directory therefore allowed access only to its owner.

Although the file itself had appropriate ownership and permissions, access to the file first required traversal permission on its parent directory.

### Resolution

No permission change was required because the restrictive permission was intentional.

The behavior was documented as part of the permissions experiment.

Administrative access could still be performed when required using `sudo`.

### Conclusion

File permissions alone do not determine whether a path can be accessed. Every parent directory in the path must also provide the necessary traversal permission.

---

## Group Membership Removal and Re-addition

### Problem

The `labuser` account was initially added to the `labadmins` supplementary group:

```bash
sudo usermod -aG labadmins labuser
```

The membership was then intentionally removed for group-management testing:

```bash
sudo gpasswd -d labuser labadmins
```

After removal, the user no longer appeared in the supplementary group list.

### Verification

The membership was checked with:

```bash
id labuser
```

and:

```bash
getent group labadmins
```

The output confirmed that `labuser` was no longer a member of `labadmins`.

### Resolution

The user was added back to the group for subsequent permission testing:

```bash
sudo usermod -aG labadmins labuser
```

The result was verified with:

```bash
id labuser
```

which showed:

```text
groups=1002(labuser),1001(labadmins)
```

### Conclusion

The group-management commands behaved as expected. The incident demonstrated the difference between primary and supplementary group membership and provided a verification point for user/group administration.

---

## Shared Directory Access Testing

### Problem

A shared directory was created for testing group-based permissions:

```bash
sudo mkdir /tmp/linux-permissions-lab
sudo chown labuser:labadmins /tmp/linux-permissions-lab
sudo chmod 770 /tmp/linux-permissions-lab
```

The directory initially allowed access to the owner and members of `labadmins`, but not to `alan`.

### Investigation

The directory permissions were:

```text
drwxrwx---
```

The test file inside it was configured with:

```text
-rw-rw----
```

Because `alan` was not a member of `labadmins`, the directory did not provide him with the required permissions.

Even temporarily adding read permission to the file was insufficient because the directory itself did not provide traversal permission.

### Resolution

Directory traversal behavior was tested by temporarily changing the directory permissions:

```bash
sudo chmod o+x /tmp/linux-permissions-lab
```

This allowed `alan` to traverse the directory and read the file where the file permissions permitted it.

The directory permissions were then restored:

```bash
sudo chmod o-rx /tmp/linux-permissions-lab
```

The final intended directory configuration was:

```text
drwxrwx---
```

### Conclusion

The test demonstrated that access to a file depends on both the file's permissions and the permissions of its parent directories.

---

# Sudo and Administrative Privileges

## Restricted Access to `/etc/shadow`

### Problem

As the normal `alan` user, an attempt was made to read:

```bash
cat /etc/shadow
```

The operation was denied.

### Investigation

The permissions of the sensitive account database were checked:

```bash
ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

`/etc/shadow` was restricted to the `root` user and the `shadow` group.

The normal user therefore did not have permission to read it directly.

### Resolution

The file was accessed through the user's administrative privileges:

```bash
sudo cat /etc/shadow
```

This succeeded because `alan` was a member of the `sudo` group and had unrestricted administrative privileges according to:

```bash
sudo -l
```

### Conclusion

The behavior was expected and confirmed that Linux protects sensitive authentication information through file permissions and privilege separation.

Sensitive contents of `/etc/shadow` were not included in project documentation.

---

## Missing `/var/log/auth.log`

### Problem

While investigating sudo activity, the traditional authentication log path was checked:

```text
/var/log/auth.log
```

The file was not present on the installed Debian system.

### Investigation

Sudo activity was instead inspected through the systemd journal:

```bash
sudo journalctl -t sudo --no-pager -n 20
```

This successfully displayed recent sudo-related activity.

### Resolution

No log file was created manually. The systemd journal was used as the appropriate source for the available sudo activity.

### Conclusion

The absence of `/var/log/auth.log` did not indicate that sudo logging was broken. The system's journal provided the relevant administrative activity.

---

# Process Management

## `pstree` Command Not Available

### Problem

During process hierarchy investigation, the following command was attempted:

```bash
pstree -p
```

The command was not available on the base system.

### Investigation

The process hierarchy could still be inspected using standard tools such as:

```bash
ps -ef
```

and:

```bash
ps -o pid,ppid,user,stat,cmd -p $$
```

The process tree relationship could therefore be examined using PID and PPID information.

### Resolution

The absence of `pstree` did not prevent completion of the process-management exercises.

The installed process-management tools were sufficient for the required verification.

### Conclusion

Not every commonly used Linux administration command is necessarily installed on a minimal server installation.

Command availability should be verified before assuming a utility exists.

---

# Package Management

## Removed Package Still Resolved by Bash

### Problem

The `tree` package was installed as part of the package-management exercises:

```bash
sudo apt install tree
```

After testing it, the package was removed:

```bash
sudo apt remove tree
```

However, immediately afterward:

```bash
command -v tree
```

still returned:

```text
/usr/bin/tree
```

This appeared to indicate that the executable had not been removed.

### Investigation

The executable and package database were checked:

```bash
ls -l /usr/bin/tree
dpkg -S /usr/bin/tree
dpkg -l tree
```

The package and executable had actually been removed.

The unexpected result from:

```bash
command -v tree
```

was caused by Bash retaining the previously resolved command path in its command hash table.

### Resolution

The Bash command hash was cleared:

```bash
hash -r
```

The command was then checked again:

```bash
command -v tree
```

No path was returned.

The executable was also confirmed to be absent:

```bash
ls -l /usr/bin/tree
```

and the package database no longer contained the package:

```bash
dpkg -l tree
```

### Conclusion

The package had been removed correctly. The apparent persistence of the command was caused by Bash's cached command lookup.

### Lesson

After installing or removing executables during package-management testing, Bash's command hash may need to be refreshed with:

```bash
hash -r
```

---

# Lessons Learned

## 1. Virtual filesystems require special consideration

Filesystems such as `/proc` are dynamic and may change while administrative tools are traversing them.

## 2. Directory permissions are as important as file permissions

A user may have sufficient permissions on a file but still be unable to access it if one of its parent directories prevents traversal.

## 3. User, group and permission changes should always be verified

Commands such as:

```bash
id
getent group
ls -l
```

provide direct confirmation of the resulting system state.

## 4. Administrative privileges should be used deliberately

`sudo` provides access to protected system resources without requiring the normal user to operate permanently as `root`.

## 5. Systemd journal can provide administrative activity logs

The absence of a traditional `/var/log/auth.log` file does not necessarily mean that administrative activity cannot be inspected.

## 6. Minimal server installations may not contain every utility

Tools such as `pstree` may not be installed by default. Standard utilities such as `ps` can often provide equivalent diagnostic information.

## 7. Shell state can affect command verification

Bash may cache executable paths. After package removal, `hash -r` can be required before command lookup accurately reflects the filesystem.

---

# Status

**Resolved / Documented**

All troubleshooting encountered during Phase 2 was resolved or verified as expected Linux behavior. The Phase 2 system administration exercises were subsequently completed successfully.