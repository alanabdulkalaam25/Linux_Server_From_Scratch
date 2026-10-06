# Phase 2 — Linux System Administration

## Objective

The objective of this phase is to develop practical Linux system administration skills by inspecting and configuring the Debian server at the operating-system level.

This phase focuses on the Linux filesystem hierarchy, users and groups, ownership and permissions, privilege management, processes, package management, environment variables, and shell configuration.

---

## Environment

| Component        | Configuration                   |
| ---------------- | ------------------------------- |
| Operating System | Debian GNU/Linux 13 (trixie)    |
| Hostname         | `DEB-SERVER`                    |
| Architecture     | x86-64                          |
| Hypervisor       | VMware Workstation Pro          |
| Purpose          | Linux server administration lab |

---

## Phase Scope

This phase covers:

- Linux filesystem hierarchy
- Users and groups
- User and group management
- File ownership
- Linux permissions
- `sudo` and administrative privileges
- Process management
- Package management
- Environment variables
- Shell configuration

---

## 2.1 Linux Filesystem Hierarchy

### Objective

The objective of this section is to inspect and understand the Linux filesystem hierarchy used by the Debian server.

The investigation focuses on the purpose of major filesystem directories, the distinction between persistent and virtual filesystems, filesystem mounts, and current disk usage.

### Initial Identity

The current administrative session was verified using:

```bash
whoami
hostname
pwd
```

The server reported:

```text
User:     alan
Hostname: DEB-SERVER
Home:     /home/alan
```

### Filesystem Hierarchy Investigation

The root filesystem was inspected using:

```bash
ls -lah /
```

The major system directories were then examined using:

```bash
ls -ld /boot /dev /etc /home /opt /proc /root /run /srv /sys /tmp /usr /var
```

The Debian installation follows the standard Linux filesystem hierarchy.

| Directory | Purpose                                                                 |
| --------- | ----------------------------------------------------------------------- |
| `/`       | Root of the entire filesystem hierarchy                                 |
| `/boot`   | Bootloader and kernel-related files                                     |
| `/dev`    | Device files exposed to the operating system                            |
| `/etc`    | System-wide configuration files                                         |
| `/home`   | Home directories of normal users                                        |
| `/opt`    | Optional/additional application software                                |
| `/proc`   | Virtual filesystem containing process and kernel information            |
| `/root`   | Home directory of the root user                                         |
| `/run`    | Runtime data created by the system and services                         |
| `/srv`    | Data intended to be served by system services                           |
| `/sys`    | Virtual filesystem exposing kernel and device information               |
| `/tmp`    | Temporary files                                                         |
| `/usr`    | User-space applications, libraries, utilities, and shared data          |
| `/var`    | Variable data such as logs, package information, caches, and spool data |

### Merged `/usr` Layout

The root directory contains several symbolic links:

```text
/bin    -> /usr/bin
/lib    -> /usr/lib
/lib64   -> /usr/lib64
/sbin   -> /usr/sbin
```

This indicates that the Debian installation uses the modern merged `/usr` filesystem layout.

Instead of maintaining completely separate directory trees for these locations, the system uses `/usr` as the primary location and provides symbolic links for compatibility with traditional filesystem paths.

### Mounted Filesystems

The mounted filesystem hierarchy was inspected using:

```bash
findmnt
```

The primary persistent filesystem is:

```text
/dev/sda2
```

mounted at:

```text
/
```

with the `ext4` filesystem.

The EFI System Partition is:

```text
/dev/sda1
```

mounted at:

```text
/boot/efi
```

with the `vfat` filesystem.

Several directories are backed by virtual or temporary filesystems:

| Mount Point                 | Filesystem | Purpose                         |
| --------------------------- | ---------- | ------------------------------- |
| `/dev`                      | `devtmpfs` | Device management               |
| `/proc`                     | `proc`     | Process and kernel information  |
| `/sys`                      | `sysfs`    | Kernel and hardware information |
| `/run`                      | `tmpfs`    | Runtime system data             |
| `/tmp`                      | `tmpfs`    | Temporary data                  |
| `/dev/shm`                  | `tmpfs`    | Shared memory                   |
| `/sys/firmware/efi/efivars` | `efivarfs` | UEFI firmware variables         |

This demonstrates that a Linux filesystem hierarchy is not simply a collection of directories stored on one physical disk. Some directories represent kernel interfaces, device management, runtime state, or temporary storage.

### Disk Usage

Filesystem capacity was inspected using:

```bash
df -h
```

The main persistent filesystem currently reports:

```text
/dev/sda2
23 GB total
1.1 GB used
21 GB available
5% usage
```

The EFI partition reports:

```text
/dev/sda1
975 MB total
9.1 MB used
966 MB available
1% usage
```

The system also uses temporary filesystems for `/dev`, `/run`, `/dev/shm`, `/tmp`, and other runtime locations.

### Directory Usage

Persistent disk usage was examined using:

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

The main directories currently consuming disk space are:

| Directory | Approximate Usage |
| --------- | ----------------: |
| `/usr`    |            681 MB |
| `/var`    |            299 MB |
| `/boot`   |             75 MB |
| `/etc`    |            4.5 MB |
| `/root`   |             24 KB |
| `/home`   |             52 KB |
| `/opt`    |              4 KB |
| `/srv`    |              4 KB |

The total persistent usage measured at the root level is approximately:

```text
1.1 GB
```

The `/usr` directory currently accounts for the largest portion of the installed operating-system software, while `/var` contains variable system data such as logs, package databases, and caches.

### Important Directory Observations

The `/home` directory currently contains the administrative user's home directory:

```text
/home/alan
```

The `/srv` directory is currently empty apart from the directory itself. This is expected because no service has yet been deployed that uses `/srv`.

The `/opt` directory is also currently empty, as no optional third-party application has been installed there.

The `/var` directory contains several important administrative locations, including:

```text
/var/cache
/var/lib
/var/log
/var/spool
```

These directories will become more relevant as additional services and administrative tasks are introduced.

### Dynamic Filesystem Observation

The initial `du` command produced transient messages while traversing `/proc`, such as:

```text
du: cannot access '/proc/...': No such file or directory
```

These messages are expected because `/proc` is a dynamic virtual filesystem. Processes can terminate while the filesystem is being traversed, causing entries to disappear between the time they are discovered and accessed.

To avoid counting virtual filesystems as physical disk usage, the following command was used:

```bash
sudo du -xhd1 / 2>/dev/null | sort -h
```

The `-x` option restricts the calculation to the same filesystem as `/`, providing a more useful view of actual persistent disk usage.

### Findings

The Debian server uses a standard Linux filesystem hierarchy with a modern merged `/usr` layout.

The root filesystem is an `ext4` filesystem mounted from `/dev/sda2`, while the EFI System Partition is mounted at `/boot/efi`.

Several important paths such as `/proc`, `/sys`, `/dev`, `/run`, and `/tmp` are provided by virtual or temporary filesystems rather than consuming persistent storage on the root filesystem.

The majority of the currently installed persistent software occupies `/usr`, while `/var` contains the majority of variable system data.

The server remains lightly utilized, with approximately 1.1 GB of the 23 GB root filesystem currently occupied.

### Verification

The filesystem hierarchy and storage configuration were verified using:

```bash
findmnt
df -h
sudo du -xhd1 / 2>/dev/null | sort -h
```

The major system directories were also inspected individually to confirm their current contents and ownership.
```

## One correction to make in your thinking

Don't interpret:

```text
du /usr = 681M
du /var = 299M
df / = 1.1G
```

as an error because the numbers don't exactly add up. `du` measures directory contents in a different way from `df` measuring filesystem blocks, and filesystem metadata, deleted-but-open files, allocation differences, etc. can affect the comparison.

Your `df` result is the authoritative answer for **filesystem capacity**, while `du` is useful for **directory-level disk consumption**.

---


## 2.2 Users and Groups

### Objective

The objective of this section is to understand the user and group model used by the Debian server, including user identities, primary and supplementary groups, system accounts, and the account database files.

### Current User Identity

The current administrative account was inspected using:

```bash
whoami
id
groups
```

The server reported:

```text
User: alan
UID: 1000
Primary GID: 1000
Primary Group: alan
Home Directory: /home/alan
Login Shell: /bin/bash
```

The account is a non-root user and is used for normal administration of the server.

### User Account Database

The configured users were inspected using:

```bash
getent passwd
```

The system contains the root account, the administrative account `alan`, and a number of system/service accounts.

Examples of service-oriented accounts include:

```text
www-data
sshd
systemd-network
systemd-timesync
messagebus
_apt
nobody
```

These accounts are used by operating-system components and services rather than normal interactive administration.

Many of these accounts use:

```text
/usr/sbin/nologin
```

or:

```text
/bin/false
```

as their login shell, preventing normal interactive login.

### User Account Structure

The `alan` account is represented in `/etc/passwd` as:

```text
alan:x:1000:1000:Alan Abdul Kalaam,,,:/home/alan:/bin/bash
```

The fields represent:

```text
Username
Password placeholder
UID
GID
GECOS / user information
Home directory
Login shell
```

The `x` field indicates that the password information is stored separately in `/etc/shadow`.

### Group Membership

The groups associated with the `alan` account were inspected using:

```bash
groups
```

The account currently belongs to:

```text
alan
cdrom
floppy
sudo
audio
dip
video
plugdev
users
netdev
```

The complete identity information was verified using:

```bash
id alan
```

The `sudo` group is particularly important because membership allows the account to obtain administrative privileges through `sudo`, subject to the system's sudo configuration.

### Primary and Supplementary Groups

The account has:

```text
Primary group:
alan (GID 1000)
```

and the following supplementary groups:

```text
cdrom
floppy
sudo
audio
dip
video
plugdev
users
netdev
```

The primary group is associated with files created by the user by default, while supplementary groups provide additional access permissions to system resources.

### Group Database

The configured groups were inspected using:

```bash
getent group
```

The system contains standard system groups as well as groups associated with devices, services, and administrative functions.

Examples include:

```text
root
sudo
users
www-data
systemd-journal
systemd-network
netdev
_ssh
```

The `alan` account is explicitly listed as a member of groups such as:

```text
sudo
users
netdev
```

as well as several device-related groups.

### Account Database Files

The permissions and ownership of the primary account database files were checked using:

```bash
sudo ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

The server reported:

```text
-rw-r--r-- 1 root root   /etc/passwd
-rw-r----- 1 root shadow /etc/shadow
-rw-r--r-- 1 root root   /etc/group
-rw-r----- 1 root shadow /etc/gshadow
```

The exact file sizes and timestamps may change as the system is modified.

The important security properties are:

| File           | Purpose                          | Access                   |
| -------------- | -------------------------------- | ------------------------ |
| `/etc/passwd`  | User account information         | Readable by normal users |
| `/etc/shadow`  | Password and authentication data | Restricted               |
| `/etc/group`   | Group definitions and membership | Readable by normal users |
| `/etc/gshadow` | Secure group information         | Restricted               |

The contents of `/etc/shadow` and `/etc/gshadow` were not exposed or recorded in project documentation because they contain sensitive authentication information.

### Findings

The Debian server uses a standard Linux user and group model.

The primary administrative account is the non-root user `alan`, with UID `1000` and primary group `alan` with GID `1000`.

The account has membership in the `sudo` group, allowing administrative commands to be executed through `sudo`.

The system also contains multiple service accounts that are intentionally configured without interactive login shells.

The account databases are stored in the standard locations under `/etc`, with sensitive authentication information protected through restrictive permissions.

### Verification

User and group configuration was verified using:

```bash
whoami
id
groups
getent passwd
getent group
getent passwd alan
id alan
sudo ls -l /etc/passwd /etc/shadow /etc/group /etc/gshadow
```

No changes to the existing user or group configuration were required during this investigation.

---

## 2.3 User and Group Management

### Objective

The objective of this section is to practice Linux user and group management. This includes creating users and groups, assigning group memberships, verifying account information, understanding primary and supplementary groups, and removing group memberships.

User and group management is an essential part of Linux system administration because access to files, services, and administrative functions is commonly controlled through users and groups.

---

### 2.3.1 Inspecting Existing User and Group Configuration

Before modifying the system, the existing administrative configuration was inspected.

The `sudo` group was checked using:

```bash
getent group sudo
```

Result:

```text
sudo:x:27:alan
```

This confirmed that the `alan` account is a member of the `sudo` group.

The root account was inspected using:

```bash
sudo getent passwd root
```

Result:

```text
root:x:0:0:root:/root:/bin/bash
```

This shows that the root account has UID `0`, GID `0`, home directory `/root`, and `/bin/bash` as its login shell.

Password status was checked with:

```bash
sudo passwd -S alan
sudo passwd -S root
```

Both accounts were reported with status `P`, indicating that a password is set.

The administrative privileges available to the current user were verified using:

```bash
sudo -l
```

The result showed:

```text
User alan may run the following commands on DEB-SERVER:
    (ALL : ALL) ALL
```

Therefore, the `alan` account has unrestricted administrative privileges through `sudo`.

---

### 2.3.2 Creating a Management Group

A dedicated test group named `labadmins` was created:

```bash
sudo groupadd labadmins
```

The group was verified using:

```bash
getent group labadmins
```

Result:

```text
labadmins:x:1001:
```

The group was initially empty.

The numeric GID is assigned by the system and may vary depending on the existing accounts and groups.

---

### 2.3.3 Creating a Test User

A dedicated test user named `labuser` was created:

```bash
sudo useradd -m -s /bin/bash labuser
```

The options used were:

- `-m` — create the user's home directory
- `-s /bin/bash` — assign Bash as the login shell

The resulting account was verified using:

```bash
getent passwd labuser
```

Result:

```text
labuser:x:1001:1002::/home/labuser:/bin/bash
```

The account was further inspected using:

```bash
id labuser
```

Result:

```text
uid=1001(labuser) gid=1002(labuser) groups=1002(labuser)
```

The system automatically created a primary group named `labuser` with GID `1002`.

Therefore, the initial account structure was:

```text
User:
  labuser
  UID: 1001

Primary group:
  labuser
  GID: 1002
```

---

### 2.3.4 Adding a Supplementary Group

The `labuser` account was added to the `labadmins` supplementary group:

```bash
sudo usermod -aG labadmins labuser
```

The membership was verified using:

```bash
id labuser
```

Result:

```text
uid=1001(labuser) gid=1002(labuser) groups=1002(labuser),1001(labadmins)
```

The group was also checked from the group database:

```bash
getent group labadmins
```

Result:

```text
labadmins:x:1001:labuser
```

This confirmed the membership from both directions:

```text
User → Groups

labuser
├── labuser       (primary group)
└── labadmins     (supplementary group)


Group → Users

labadmins
└── labuser
```

The `-aG` option was used intentionally. The `-a` option appends the supplementary group instead of replacing the user's existing supplementary group memberships.

---

### 2.3.5 Inspecting the User's Home Directory

The home directory was inspected using:

```bash
ls -ld /home/labuser
```

Result:

```text
drwx------ 2 labuser labuser 4096 Oct  4 11:05 /home/labuser
```

The directory is owned by `labuser:labuser`.

Its permissions were:

```text
Owner  → rwx
Group  → ---
Others → ---
```

Therefore, the directory is restricted to its owner.

Running:

```bash
ls -la /home/labuser
```

as the `alan` account resulted in:

```text
ls: cannot open directory '/home/labuser': Permission denied
```

This behavior demonstrated that directory ownership and permissions affect access even when another user has administrative capabilities available through `sudo`.

---

### 2.3.6 Removing a Supplementary Group Membership

After testing the supplementary group membership, `labuser` was removed from `labadmins`:

```bash
sudo gpasswd -d labuser labadmins
```

The system reported:

```text
Removing user labuser from group labadmins
```

The result was verified with:

```bash
id labuser
```

Result:

```text
uid=1001(labuser) gid=1002(labuser) groups=1002(labuser)
```

The group was also verified:

```bash
getent group labadmins
```

Result:

```text
labadmins:x:1001:
```

The `labuser` account retained its primary group while its supplementary membership in `labadmins` was removed.

---

### 2.3.7 Final User and Group State

At the end of this exercise, the test account and group had the following configuration:

| Item                   | Value           |
| ---------------------- | --------------- |
| Test user              | `labuser`       |
| UID                    | `1001`          |
| Primary group          | `labuser`       |
| Primary GID            | `1002`          |
| Supplementary groups   | None            |
| Home directory         | `/home/labuser` |
| Login shell            | `/bin/bash`     |
| Test group             | `labadmins`     |
| Test group GID         | `1001`          |
| Members of `labadmins` | None            |

The test user and group were intentionally retained because they will be used in subsequent sections to demonstrate file ownership and Linux permissions.

---

### Findings

The following concepts were demonstrated practically:

1. Linux users are identified internally using numeric UIDs.
2. Groups are identified using numeric GIDs.
3. A user has one primary group and can belong to multiple supplementary groups.
4. `getent passwd` can be used to retrieve user account information.
5. `getent group` can be used to retrieve group information.
6. `id` provides a convenient summary of a user's UID, GID, and group memberships.
7. `useradd` can create a user account and its home directory.
8. `groupadd` creates a new group.
9. `usermod -aG` adds a user to supplementary groups without replacing existing memberships.
10. `gpasswd -d` removes a user from a supplementary group.
11. User and group membership directly affects access to system resources.
12. Directory permissions can prevent access to a user's home directory even when the directory exists and is correctly configured.

---

### Verification

The final configuration was verified using:

```bash
getent passwd labuser
getent group labuser
getent group labadmins
id labuser
ls -ld /home/labuser
```

The results confirmed that the user, primary group, supplementary group membership, home directory, and ownership were configured as expected.

### Result

User and group management was successfully configured and verified on the Debian server. A dedicated test user and group were created, modified, inspected, and used to demonstrate Linux account and group-management behavior.

The test account and group will be reused in the following sections for practical demonstrations of file ownership and permissions.

---

## 2.4 File Ownership

### Objective

The objective of this section is to understand Linux file ownership and practice changing the owner and group associated with files.

Every file and directory in Linux has an associated user owner and group owner. These ownership attributes work together with Linux permission bits to control access to filesystem resources.

---

### 2.4.1 Inspecting Existing File Ownership

Existing system directories and files were inspected using the `ls` command:

```bash
ls -ld /home/alan
ls -ld /home/labuser
ls -ld /tmp
ls -ld /etc

ls -l /etc/passwd
ls -l /etc/group
ls -l /etc/hostname
```

The long listing format displays information including:

```text
permissions  links  owner  group  size  date  name
```

For example:

```text
-rw-r--r-- 1 root root ... /etc/hostname
```

indicates that:

```text
Owner       → root
Group owner → root
```

This demonstrates that Linux maintains both a user owner and a group owner for filesystem objects.

---

### 2.4.2 Creating a Test File

A test file was created inside the `labuser` home directory:

```bash
sudo touch /home/labuser/ownership-test.txt
```

The resulting ownership was inspected using:

```bash
ls -l /home/labuser/ownership-test.txt
```

Because the `touch` command was executed through `sudo`, the file was initially created with `root` as its owner.

The initial ownership was:

```text
Owner       → root
Group owner → root
```

This demonstrates that file ownership is determined by the effective user and group under which the file is created.

---

### 2.4.3 Changing the Group Owner

The group owner was changed from `root` to the previously created `labadmins` group:

```bash
sudo chown :labadmins /home/labuser/ownership-test.txt
```

The ownership was then verified.

The resulting ownership was:

```text
Owner       → root
Group owner → labadmins
```

The `chown` syntax:

```bash
chown :GROUP FILE
```

changes only the group owner.

---

### 2.4.4 Changing the User Owner

The owner and group were then explicitly configured as `labuser` and `labadmins`:

```bash
sudo chown labuser:labadmins /home/labuser/ownership-test.txt
```

The resulting listing confirmed:

```text
-rw-r--r-- 1 labuser labadmins 0 Oct 4 17:32 ownership-test.txt
```

Therefore:

```text
Owner       → labuser
Group owner → labadmins
```

The following `chown` forms were demonstrated:

```bash
chown USER FILE
chown :GROUP FILE
chown USER:GROUP FILE
```

Their purposes are:

| Command                        | Operation                   |
| ------------------------------ | --------------------------- |
| `chown labuser file`           | Change the user owner       |
| `chown :labadmins file`        | Change only the group owner |
| `chown labuser:labadmins file` | Change both owner and group |

---

### 2.4.5 Verifying Numeric Ownership

Linux internally represents users and groups using numeric UIDs and GIDs.

The numeric ownership of the test file was inspected using:

```bash
ls -ln /home/labuser/ownership-test.txt
```

The result was:

```text
-rw-r--r-- 1 1001 1001 0 Oct 4 17:32 ownership-test.txt
```

This corresponds to:

```text
UID 1001 → labuser
GID 1001 → labadmins
```

The relationship was confirmed using:

```bash
getent passwd labuser
getent group labadmins
```

The results were:

```text
labuser:x:1001:1002::/home/labuser:/bin/bash
labadmins:x:1001:
```

Therefore, the numeric UID/GID values observed in the file metadata could be mapped back to their corresponding user and group names.

---

### 2.4.6 Directory Access Observation

The test file was located inside:

```text
/home/labuser
```

The directory permissions were:

```text
drwx------ 2 labuser labuser ... /home/labuser
```

Because the directory grants access only to its owner, the `alan` account could not directly access the file despite the file itself having readable permissions.

The following commands therefore produced `Permission denied` when executed as `alan`:

```bash
ls -l /home/labuser/ownership-test.txt
ls -ln /home/labuser/ownership-test.txt
stat /home/labuser/ownership-test.txt
```

This demonstrates an important Linux filesystem principle:

> Access to a file also depends on the permissions of its parent directories. Having read permission on a file does not necessarily allow a user to reach that file if the user cannot traverse the directory containing it.

Administrative access through `sudo` was used when inspecting the directory and file during the ownership exercise.

---

### Findings

The following concepts were demonstrated:

1. Every Linux file has a user owner and a group owner.
2. File ownership can be inspected using `ls -l`.
3. Numeric ownership can be inspected using `ls -ln`.
4. `chown` can change file ownership.
5. `chown :GROUP` changes only the group owner.
6. `chown USER:GROUP` changes both the user and group owner.
7. Linux internally associates ownership with numeric UIDs and GIDs.
8. User and group names are resolved from the system's account and group databases.
9. File permissions alone do not determine whether a path can be accessed.
10. Parent directory permissions can prevent access to a file even when the file itself has readable permissions.

---

### Verification

The final ownership of the test file was verified using administrative access:

```bash
sudo ls -l /home/labuser/ownership-test.txt
```

The resulting ownership was:

```text
labuser:labadmins
```

The numeric representation was:

```text
UID 1001
GID 1001
```

These values were confirmed using:

```bash
getent passwd labuser
getent group labadmins
```

---

### Result

File ownership was successfully inspected and modified on the Debian server.

A test file was created, its original `root:root` ownership was observed, and ownership was subsequently changed to:

```text
labuser:labadmins
```

The exercise also demonstrated the relationship between file ownership, numeric UIDs/GIDs, and directory access permissions.

The test file `ownership-test.txt` and the `labuser`/`labadmins` accounts will be retained for the next section, **2.5 — Linux Permissions**, where ownership and permission bits will be tested together.

---

## 2.5 Linux Permissions

### Objective

The objective of this section is to understand the Linux permission model and practically demonstrate how read, write, and execute permissions control access to files and directories.

The exercise covered:

- Permission ownership categories
- Read (`r`), write (`w`), and execute (`x`) permissions
- Owner, group, and others
- Symbolic permission modification using `chmod`
- Numeric/octal permission notation
- File permissions
- Directory permissions
- The relationship between parent-directory permissions and file access

---

### 2.5.1 Linux Permission Model

Linux permissions are divided into three access categories:

```text
Owner
Group
Others
```

Each category can have three basic permissions:

```text
r → read
w → write
x → execute
```

For regular files:

| Permission | Meaning                         |
| ---------- | ------------------------------- |
| `r`        | Read the contents of the file   |
| `w`        | Modify the contents of the file |
| `x`        | Execute the file                |

For directories:

| Permission | Meaning                                     |
| ---------- | ------------------------------------------- |
| `r`        | List directory contents                     |
| `w`        | Create, delete, or rename directory entries |
| `x`        | Enter or traverse the directory             |

---

### 2.5.2 Reading Permission Strings

The test file initially had the following permissions:

```text
-rw-r--r--
```

The first character represents the file type:

```text
- → regular file
d → directory
l → symbolic link
```

The remaining nine characters are divided into three groups:

```text
-rw-r--r--
 │  │  │
 │  │  └── Others
 │  └───── Group
 └──────── Owner
```

Therefore:

```text
Owner   → rw-
Group   → r--
Others  → r--
```

This configuration is equivalent to octal permission `644`.

---

### 2.5.3 Testing Owner Permissions

The existing file was:

```text
/home/labuser/ownership-test.txt
```

with ownership:

```text
labuser:labadmins
```

The file initially had:

```text
-rw-r--r--
```

The `labuser` account was used to test owner permissions.

As the owner, `labuser` successfully wrote to the file:

```bash
echo "Linux permissions test" > /home/labuser/ownership-test.txt
```

The contents were then verified using:

```bash
cat /home/labuser/ownership-test.txt
```

The owner write permission was subsequently removed:

```bash
chmod u-w /home/labuser/ownership-test.txt
```

The resulting permissions were:

```text
-r--r--r--
```

When `labuser` attempted to write to the file again:

```bash
echo "This should fail" > /home/labuser/ownership-test.txt
```

the operation failed with:

```text
Permission denied
```

This demonstrated that the file owner is still restricted by the permission bits assigned to the owner category.

Owner write permission was restored using:

```bash
chmod u+w /home/labuser/ownership-test.txt
```

The file was returned to:

```text
-rw-r--r--
```

---

### 2.5.4 Testing Group Permissions

A separate user named `labguest` was created and added to the `labadmins` group.

The membership was verified using:

```bash
id labguest
```

which showed:

```text
uid=1002(labguest) gid=1003(labguest) groups=1003(labguest),1001(labadmins)
```

The original file could not be used directly for the group test because its parent directory had restrictive permissions:

```text
drwx------ 2 labuser labuser ... /home/labuser
```

Therefore, a separate controlled permissions directory was created:

```bash
sudo mkdir /tmp/linux-permissions-lab
sudo chown labuser:labadmins /tmp/linux-permissions-lab
sudo chmod 770 /tmp/linux-permissions-lab
```

The resulting directory permissions were:

```text
drwxrwx---
```

A test file was created inside the directory:

```bash
echo "Group permissions test" > /tmp/linux-permissions-lab/group-test.txt
```

Its group ownership was changed to `labadmins`:

```bash
chown :labadmins /tmp/linux-permissions-lab/group-test.txt
```

The file permissions were then set to:

```bash
chmod 660 /tmp/linux-permissions-lab/group-test.txt
```

The resulting configuration was:

```text
-rw-rw---- 1 labuser labadmins ... group-test.txt
```

Therefore:

```text
Owner   → rw-
Group   → rw-
Others  → ---
```

As a member of `labadmins`, `labguest` was able to read the file:

```bash
cat /tmp/linux-permissions-lab/group-test.txt
```

and successfully append data:

```bash
echo "Written by labguest" >> /tmp/linux-permissions-lab/group-test.txt
```

The final contents were:

```text
Group permissions test
Written by labguest
```

This demonstrated that group members receive the permissions assigned to the file's group category.

---

### 2.5.5 Testing Permissions for Others

The `alan` account was used as an example of an "other" user because it was neither the owner of the test file nor a member of `labadmins`.

The test file initially had:

```text
-rw-rw----
```

Therefore, `alan` had no permissions through the `others` category.

Attempts to read or write the file failed with:

```text
Permission denied
```

The file was then given read permission for others:

```bash
sudo chmod o+r /tmp/linux-permissions-lab/group-test.txt
```

The resulting permissions were:

```text
-rw-rw-r--
```

However, `alan` initially still could not access the file because the parent directory was:

```text
drwxrwx---
```

and therefore provided no permissions to others.

To demonstrate directory traversal, execute permission was temporarily added for others:

```bash
sudo chmod o+x /tmp/linux-permissions-lab
```

The directory became:

```text
drwxrwx--x
```

`alan` could then successfully read the file because:

```text
Directory → x
File      → r
```

However, writing still failed because the file provided only read permission to others.

After the test, the directory's original restricted permissions were restored:

```bash
sudo chmod o-rx /tmp/linux-permissions-lab
```

resulting in:

```text
drwxrwx---
```

---

### 2.5.6 Directory Execute Permission

The experiments demonstrated an important difference between file and directory permissions.

For a directory:

```text
r → list directory contents
w → create/delete/rename entries
x → enter/traverse the directory
```

The test directory:

```text
/tmp/linux-permissions-lab
```

was initially configured as:

```text
drwxrwx---
```

When `alan` had no `x` permission on the directory, he could not access files inside it even when those files themselves granted permissions to others.

After temporarily adding:

```bash
sudo chmod o+x /tmp/linux-permissions-lab
```

the directory became:

```text
drwxrwx--x
```

This allowed `alan` to traverse the directory and access a file according to the file's own permissions.

This demonstrated that access to a file depends not only on the file's permission bits but also on the permissions of its parent directories.

---

### 2.5.7 Numeric Permission Notation

Linux permissions can also be represented using octal values.

The permission values are:

```text
r = 4
w = 2
x = 1
```

The values are combined for each permission category:

| Permission | Value |
| ---------- | ----: |
| `---`      |     0 |
| `--x`      |     1 |
| `-w-`      |     2 |
| `-wx`      |     3 |
| `r--`      |     4 |
| `r-x`      |     5 |
| `rw-`      |     6 |
| `rwx`      |     7 |

For example:

```text
rw-r--r--
```

becomes:

```text
6 4 4
```

or:

```text
644
```

because:

```text
Owner   → rw- = 4 + 2 = 6
Group   → r-- = 4
Others  → r-- = 4
```

---

### 2.5.8 Testing Common Numeric Modes

The test file was assigned several numeric permission modes.

#### Mode 600

```bash
sudo chmod 600 /tmp/linux-permissions-lab/group-test.txt
```

Result:

```text
-rw-------
```

Meaning:

```text
Owner   → rw-
Group   → ---
Others  → ---
```

#### Mode 640

```bash
sudo chmod 640 /tmp/linux-permissions-lab/group-test.txt
```

Result:

```text
-rw-r-----
```

Meaning:

```text
Owner   → rw-
Group   → r--
Others  → ---
```

#### Mode 644

The file was finally restored to:

```bash
sudo chmod 644 /tmp/linux-permissions-lab/group-test.txt
```

Result:

```text
-rw-r--r--
```

Meaning:

```text
Owner   → rw-
Group   → r--
Others  → r--
```

These tests demonstrated how octal values directly map to Linux permission bits.

---

### 2.5.9 Final Test Environment

The temporary permissions laboratory was restored to controlled permissions after the experiments.

Directory:

```text
/tmp/linux-permissions-lab
```

Ownership:

```text
labuser:labadmins
```

Permissions:

```text
drwxrwx---
```

Test file:

```text
/tmp/linux-permissions-lab/group-test.txt
```

Ownership:

```text
labuser:labadmins
```

Permissions:

```text
-rw-r--r--
```

The original ownership test file remained available:

```text
/home/labuser/ownership-test.txt
```

with ownership:

```text
labuser:labadmins
```

---

### Findings

The following concepts were demonstrated practically:

1. Linux permissions are divided into owner, group, and others.
2. The three basic permissions are read (`r`), write (`w`), and execute (`x`).
3. File and directory permissions have different meanings.
4. File owners are still restricted by the permissions assigned to the owner category.
5. Group membership allows users to receive the permissions assigned to the file's group.
6. Users outside the owner and group categories are evaluated under the `others` permissions.
7. `chmod` can modify permissions symbolically or numerically.
8. `u`, `g`, and `o` represent user/owner, group, and others.
9. Numeric permission values are based on `r=4`, `w=2`, and `x=1`.
10. Parent directory permissions affect access to files contained within the directory.
11. Directory execute permission is required to traverse a directory.
12. File permissions alone do not guarantee access if a parent directory prevents traversal.

---

### Verification

The final test environment was verified using:

```bash
sudo ls -ld /tmp/linux-permissions-lab
sudo ls -l /tmp/linux-permissions-lab/group-test.txt
sudo ls -l /home/labuser/ownership-test.txt
whoami
```

The final active administrative user was verified as:

```text
alan
```

The permission laboratory was restored to:

```text
Directory:
drwxrwx--- labuser:labadmins

File:
-rw-r--r-- labuser:labadmins
```

---

### Result

Linux file and directory permissions were successfully configured, modified, tested, and verified.

The exercises demonstrated the practical relationship between ownership, groups, permission bits, numeric permission modes, and directory traversal.

The resulting knowledge will be used in subsequent sections involving `sudo`, administrative privileges, processes, and system administration.

---

## 2.6 sudo and Administrative Privileges

### Objective

The objective of this section is to understand how administrative privileges are managed on a Linux server using `sudo`.

The practical exercises demonstrate:

- Normal user privileges
- Membership in the `sudo` group
- Inspection of sudo privileges
- Temporary privilege escalation
- The difference between a normal user and the root user
- Access to root-owned resources
- Sudo security defaults
- Administrative activity logging

The principle of least privilege was also considered by using `sudo` for individual administrative operations rather than continuously operating from a root shell.

---

### 2.6.1 Current User Identity

The current user's identity was verified using:

```bash
whoami
id
```

The result showed:

```text
alan
uid=1000(alan) gid=1000(alan)
```

The `alan` account is a normal user account rather than the root account.

The account is also a member of several supplementary groups, including:

```text
sudo
```

This membership provides administrative privileges through the `sudo` mechanism.

---

### 2.6.2 Inspecting the sudo Group

The system's `sudo` group was inspected using:

```bash
getent group sudo
```

Result:

```text
sudo:x:27:alan
```

This confirms that the `alan` account is a member of the `sudo` group.

On Debian systems, membership in the appropriate administrative group can allow a user to execute authorized commands with elevated privileges.

---

### 2.6.3 Inspecting sudo Privileges

The effective sudo privileges of the current user were inspected using:

```bash
sudo -l
```

The relevant result was:

```text
User alan may run the following commands on DEB-SERVER:
    (ALL : ALL) ALL
```

This configuration means that `alan` is authorized to execute commands:

```text
As any user
As any group
For any command
```

Therefore, the account currently has full administrative capability through `sudo`.

The command also displayed the following sudo defaults:

```text
env_reset
mail_badpass
secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
use_pty
```

These defaults provide additional controls around the execution environment and privileged command execution.

---

### 2.6.4 Testing Access to Protected Files

The `/etc/shadow` file was used to demonstrate the difference between normal and elevated privileges.

First, the file was accessed without `sudo`:

```bash
cat /etc/shadow
```

The operation failed:

```text
Permission denied
```

The same operation was then performed with `sudo`:

```bash
sudo cat /etc/shadow
```

The operation succeeded because the command was executed with root privileges.

The contents of `/etc/shadow` were intentionally not recorded in this documentation because the file contains sensitive password-hash information.

This demonstrated:

```text
Normal execution:
alan → insufficient privileges

sudo execution:
alan → root privileges → access granted
```

---

### 2.6.5 Comparing Normal and Elevated Identity

The difference between the normal user and the effective privileged user was demonstrated using:

```bash
whoami
sudo whoami
```

The results were:

```text
alan
root
```

The UID difference was also verified:

```bash
id
sudo id
```

Normal execution returned:

```text
uid=1000(alan) gid=1000(alan)
```

while the elevated command returned:

```text
uid=0(root) gid=0(root) groups=0(root)
```

This demonstrates that `sudo` does not permanently change the identity of the `alan` account. Instead, it executes the specified command with the privileges of the authorized target user, which in this case was `root`.

---

### 2.6.6 Accessing the Root Home Directory

The `/root` directory was used to demonstrate access restrictions.

Without elevated privileges:

```bash
ls -la /root
```

returned:

```text
Permission denied
```

The same operation using `sudo`:

```bash
sudo ls -la /root
```

succeeded.

The directory was shown to be owned by `root:root` with restrictive permissions:

```text
drwx------ 3 root root ... /root
```

This demonstrates that root-owned resources can be inaccessible to normal users while remaining accessible through authorized administrative privileges.

---

### 2.6.7 Sudo Version

The installed sudo version was checked using:

```bash
sudo -V | head -25
```

The server reported:

```text
Sudo version 1.9.16p2
Sudoers policy plugin version 1.9.16p2
Sudoers file grammar version 50
Sudoers I/O plugin version 1.9.16p2
Sudoers audit plugin version 1.9.16p2
```

This confirms the version of the sudo implementation currently installed on the server.

---

### 2.6.8 Sudo Activity Logging

Administrative activity was inspected using the system journal:

```bash
sudo journalctl -t sudo --no-pager -n 20
```

The journal contained sudo events including:

```text
alan : TTY=pts/1 ; PWD=/home/alan ; USER=root ; COMMAND=/usr/bin/cat /etc/shadow
```

and:

```text
alan : TTY=pts/1 ; PWD=/home/alan ; USER=root ; COMMAND=/usr/bin/whoami
```

These records demonstrate that sudo operations are logged with information such as:

- invoking user
- terminal
- working directory
- target user
- executed command

The traditional `/var/log/auth.log` file was also checked:

```bash
sudo grep sudo /var/log/auth.log | tail -20
```

but the file did not exist on this Debian installation.

The system journal therefore serves as the available source for inspecting sudo activity in the current configuration.

---

### 2.6.9 Administrative Privilege Model

The server currently follows this privilege model:

```text
Normal user
    │
    │ member of sudo group
    ▼
sudo authorization
    │
    ▼
Privileged command execution
    │
    ▼
root privileges (UID 0)
```

The normal account remains:

```text
alan
UID 1000
```

while commands executed through:

```bash
sudo command
```

can run with:

```text
UID 0
```

when authorized by the sudo policy.

---

### Findings

The following concepts were demonstrated practically:

1. `alan` is a normal user with UID `1000`.
2. `alan` is a member of the `sudo` group.
3. `sudo -l` can be used to inspect the commands a user is authorized to execute.
4. The current configuration grants `alan` unrestricted sudo access.
5. `sudo` allows individual commands to execute with elevated privileges.
6. The normal user identity remains unchanged when `sudo` is used.
7. Root has UID `0`.
8. Protected resources such as `/etc/shadow` and `/root` cannot normally be accessed by `alan`.
9. The same resources can be accessed through authorized sudo operations.
10. Sudo activity is recorded in the system journal.
11. The traditional `/var/log/auth.log` file is not present on this installation.
12. Using `sudo` for individual commands supports a least-privilege administrative workflow compared with continuously operating as root.

---

### Verification

The following commands were used to verify the administrative privilege configuration:

```bash
whoami
id
getent group sudo
sudo -l
sudo whoami
sudo id
sudo ls -la /root
sudo journalctl -t sudo --no-pager -n 20
```

The results confirmed that:

- the normal account is `alan`
- the account has UID `1000`
- the account belongs to the `sudo` group
- unrestricted sudo privileges are available
- privileged commands execute with UID `0`
- sudo activity is recorded by the system journal

---

### Result

The sudo and administrative privilege configuration of the Debian server was successfully inspected and verified.

The practical exercises demonstrated the transition between normal user privileges and temporary administrative privileges, access control for root-owned resources, sudo policy inspection, and administrative activity logging.

No modifications were made to the sudo policy or `/etc/sudoers` during this exercise.

---

## 2.7 Process Management

### Objective

The objective of this section was to understand how Linux manages running processes and to practice monitoring, inspecting, controlling, and terminating processes.

The following areas were covered:

- Process identification using PIDs and PPIDs
- Process listing and inspection
- Parent-child process relationships
- Background processes and job control
- Process states
- Process signals
- Process termination
- CPU and memory usage monitoring
- PID 1 and `systemd`
- SSH session and shell process hierarchy

---

### 2.7.1 Process Listing and Inspection

Basic process information was inspected using `ps`:

```bash
ps
ps -f
ps -u alan
ps aux
ps -ef
```

`ps` displayed processes associated with the current terminal, while `ps -f` provided additional information such as the process ID, parent process ID, user, and command.

`ps -u alan` was used to display processes owned by the `alan` user.

`ps aux` and `ps -ef` provided broader views of processes running across the system.

The process list showed both user processes and system processes. Kernel threads were displayed using names enclosed in square brackets, while normal userspace processes showed their executable commands.

---

### 2.7.2 PID and PPID

The current shell process was identified using:

```bash
echo $$
```

Output:

```text
1292
```

The shell was then inspected with:

```bash
ps -o pid,ppid,user,stat,cmd -p $$
```

Result:

```text
PID    PPID USER  STAT CMD
1292   1291 alan  Ss   -bash
```

This established that:

- PID `1292` is the current Bash shell.
- PPID `1291` is its parent SSH session.
- The shell is running as user `alan`.
- `Ss` indicates a sleeping session-leader process.

The parent-child relationship was further verified with:

```bash
echo "Shell PID: $$"
echo "Parent PID: $PPID"

ps -o pid,ppid,user,stat,cmd -p $$,$PPID
```

Result:

```text
PID    PPID USER  STAT CMD
1291   1270 alan  S    sshd-session: alan@pts/0
1292   1291 alan  Ss   -bash
```

This demonstrated the relationship:

```text
sshd-session
    └── bash
```

---

### 2.7.3 Process Tree

The `pstree` command was checked:

```bash
pstree -p
```

However, `pstree` was not installed on the server:

```text
-bash: pstree: command not found
```

The process hierarchy was therefore inspected using `ps` and `systemctl` instead.

No additional package was installed solely for this demonstration.

---

### 2.7.4 Background Processes and Job Control

A controlled background process was created using:

```bash
sleep 300 &
```

The shell assigned the process:

```text
[1] 1435
```

The process was inspected using:

```bash
jobs
jobs -l
ps -p 1435 -o pid,ppid,user,stat,cmd
```

The process appeared as:

```text
PID    PPID USER  STAT CMD
1435   1292 alan  S    sleep 300
```

This demonstrated that the background `sleep` process was a child of the current Bash shell.

The process was terminated using:

```bash
kill 1435
```

The shell then reported:

```text
[1]+  Terminated              sleep 300
```

A second background process was created using:

```bash
sleep 300 &
PID=$!
```

The special `$!` shell variable provided the PID of the most recently started background process.

---

### 2.7.5 Process States and Signals

A controlled process was used to demonstrate Linux process states and signals:

```bash
sleep 300 &
PID=$!

echo "PID: $PID"

ps -p "$PID" -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

The initial state was:

```text
PID    PPID USER  STAT %CPU %MEM CMD
1479   1292 alan  S    0.0  0.0  sleep 300
```

The process was stopped with `SIGSTOP`:

```bash
kill -STOP "$PID"
```

Its state changed to:

```text
PID    PPID USER  STAT %CPU %MEM CMD
1479   1292 alan  T    0.0  0.0  sleep 300
```

The `T` state indicated that the process had been stopped.

The process was then resumed using `SIGCONT`:

```bash
kill -CONT "$PID"
```

Its state returned to:

```text
PID    PPID USER  STAT %CPU %MEM CMD
1479   1292 alan  S    0.0  0.0  sleep 300
```

Finally, the process was terminated:

```bash
kill "$PID"
wait "$PID" 2>/dev/null
```

A subsequent process check returned no process information, confirming that PID `1479` had terminated.

This demonstrated the process lifecycle:

```text
Running
   ↓
SIGSTOP
   ↓
Stopped
   ↓
SIGCONT
   ↓
Running
   ↓
SIGTERM
   ↓
Terminated
```

---

### 2.7.6 PID 1 and systemd

PID 1 was inspected using:

```bash
ps -p 1 -o pid,ppid,user,stat,cmd
```

Result:

```text
PID  PPID USER  STAT CMD
1    0    root  Ss   /sbin/init
```

On this Debian system, `/sbin/init` is provided by `systemd`.

The system state was inspected using:

```bash
systemctl status
```

The server reported:

```text
State: running
Units: 293 loaded
Jobs: 0 queued
Failed: 0 units
systemd: 257.13-1~deb13u1
```

The output also showed system services such as:

- `cron.service`
- `dbus.service`
- `open-vm-tools.service`
- `ssh.service`
- `systemd-journald.service`
- `systemd-logind.service`
- `systemd-timesyncd.service`
- `systemd-udevd.service`

This demonstrated how `systemd` acts as PID 1 and manages system services.

---

### 2.7.7 CPU and Memory Monitoring

CPU usage was examined using:

```bash
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd --sort=-%cpu | head -15
```

Memory usage was examined using:

```bash
ps -eo pid,ppid,user,%cpu,%mem,stat,cmd --sort=-%mem | head -15
```

The results showed system kernel workers, SSH sessions, `systemd`, `vmtoolsd`, and other system processes.

The measurements confirmed that the server was under very light CPU and memory load during testing.

`htop` was also checked:

```bash
command -v htop
```

Result:

```text
/usr/bin/htop
```

Therefore, `htop` is already available for interactive process monitoring.

---

### 2.7.8 SSH and Shell Process Hierarchy

The running process list demonstrated the SSH session hierarchy:

```text
sshd
 └── sshd-session
      └── bash
```

The current shell was:

```text
PID 1292
PPID 1291
```

with the parent process being:

```text
sshd-session: alan@pts/0
```

Multiple SSH sessions were also visible during testing, demonstrating that each active SSH connection can have its own session and shell processes.

---

### Findings

The process management exercises demonstrated that:

1. Every running process has a unique PID.
2. Processes maintain parent-child relationships through PPIDs.
3. Bash can create and manage background processes.
4. The `jobs` command manages shell jobs, while `ps` provides system-level process information.
5. Process signals can control process execution.
6. `SIGSTOP` changes a running process to the stopped (`T`) state.
7. `SIGCONT` resumes a stopped process.
8. `kill` can terminate a process.
9. PID 1 is responsible for system initialization and service management through `systemd`.
10. `ps` can be used to monitor CPU and memory consumption.
11. SSH sessions create a process hierarchy leading to the user's shell.

### Verification

The following areas were successfully verified:

- Process identification using PID and PPID
- Process listing with `ps`
- Background job management
- Process termination
- Process state transitions
- Signal handling
- PID 1 and `systemd`
- CPU and memory process monitoring
- SSH-to-shell process hierarchy
- Availability of `htop`

### Result

Process management was successfully configured and tested on the Debian server. The practical exercises provided hands-on experience with process identification, monitoring, job control, signals, process states, resource usage, and `systemd` process management.

---

## 2.8 Package Management

### Objective

The objective of this section was to understand and practice package management on Debian Linux using `apt`, `apt-get`, and `dpkg`.

The following package-management operations were performed:

- Identifying the available package-management tools
- Inspecting APT repository configuration
- Updating package metadata
- Checking available package upgrades
- Searching for packages
- Inspecting package information
- Installing a package
- Verifying an installed package
- Inspecting package files
- Checking package status and version information
- Removing a package
- Verifying complete package removal
- Inspecting APT package priorities and metadata statistics

---

### 2.8.1 Identifying Package Management Tools

The available package-management utilities were identified using:

```bash
command -v apt
command -v apt-get
command -v dpkg
apt --version
dpkg --version | head -1
```

The system returned:

```text
/usr/bin/apt
/usr/bin/apt-get
/usr/bin/dpkg
apt 3.0.3 (amd64)
Debian 'dpkg' package management program version 1.22.22 (amd64).
```

This confirms that the system uses the Debian package-management ecosystem:

- `apt` — high-level package management and repository operations
- `apt-get` — lower-level APT command-line interface commonly used for scripted/package operations
- `dpkg` — low-level Debian package database and `.deb` package management tool

---

### 2.8.2 Inspecting APT Repositories

The configured repository sources were inspected using:

```bash
cat /etc/apt/sources.list
ls -lah /etc/apt/sources.list.d/
grep -Rhv '^[[:space:]]*#' /etc/apt/sources.list /etc/apt/sources.list.d/ 2>/dev/null
```

The active repositories were:

```text
deb http://deb.debian.org/debian/ trixie main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware

deb http://security.debian.org/debian-security trixie-security main non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware

deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware
deb-src http://deb.debian.org/debian/ trixie-updates main non-free-firmware
```

The configured repositories provide:

- **`trixie`** — main Debian package repository
- **`trixie-security`** — Debian security updates
- **`trixie-updates`** — stable updates

The `/etc/apt/sources.list.d/` directory was present but contained no additional repository configuration.

The commented CD-ROM repository entry was left disabled.

---

### 2.8.3 Updating Package Metadata

The local APT package information was refreshed using:

```bash
sudo apt update
```

The operation completed successfully and retrieved package metadata from the configured Debian repositories.

The update downloaded approximately **756 kB** of metadata.

After updating the package lists, the system reported that **6 packages could be upgraded**.

Available upgrades were inspected using:

```bash
apt list --upgradable
```

The packages identified for upgrade were:

```text
libpcre2-8-0
libpng16-16t64
libssl3t64
linux-image-amd64
openssl-provider-legacy
openssl
```

The packages were not upgraded during this section because the objective was to inspect package management rather than perform a full system upgrade.

---

### 2.8.4 Searching for Packages

The `tree` package was used as a practical package-management example.

Package searching was performed with:

```bash
apt search tree | head -20
```

APT returned matching packages, including the `tree` package.

Detailed package information was then inspected using:

```bash
apt show tree
```

The package information showed:

```text
Package: tree
Version: 2.2.1-1
Priority: optional
Section: utils
Architecture: amd64
Installed-Size: 132 kB
Depends: libc6 >= 2.38
APT-Sources: http://deb.debian.org/debian trixie/main amd64 Packages
```

This demonstrated how APT can be used to inspect a package before installation.

---

### 2.8.5 Installing a Package

The `tree` package was installed using:

```bash
sudo apt install tree
```

APT reported:

```text
0 upgraded, 1 newly installed, 0 to remove
```

The package download size was approximately **59.4 kB**, and approximately **132 kB** of disk space was required.

The installation completed successfully.

---

### 2.8.6 Verifying the Installation

The installed executable was located using:

```bash
command -v tree
```

Output:

```text
/usr/bin/tree
```

The installed version was checked using:

```bash
tree --version
```

Output:

```text
tree v2.2.1
```

The Debian package database was inspected using:

```bash
dpkg -s tree
```

The package status showed:

```text
Status: install ok installed
Version: 2.2.1-1
Architecture: amd64
```

The files installed by the package were inspected using:

```bash
dpkg -L tree
```

Important installed files included:

```text
/usr/bin/tree
/usr/share/doc/tree
/usr/share/man/man1/tree.1.gz
```

This demonstrated the difference between the package database and the actual files installed by a package.

---

### 2.8.7 Using an Installed Package

The newly installed `tree` utility was used to inspect the APT configuration directory:

```bash
tree -L 2 /etc/apt
```

The command displayed the directory structure of `/etc/apt`, including configuration files, repository sources, keyrings, and other APT-related directories.

This provided a practical demonstration that a package installation makes its executable immediately available to the system.

---

### 2.8.8 Checking Package Status and Version

The installed package state was checked using:

```bash
dpkg -l tree
```

The package was shown with the status:

```text
ii  tree  2.2.1-1  amd64
```

The `ii` status indicates that the package was installed successfully.

APT's package version information was inspected using:

```bash
apt policy tree
```

The output showed:

```text
Installed: 2.2.1-1
Candidate: 2.2.1-1
```

The candidate version matched the installed version, indicating that no newer `tree` version was available from the configured repositories.

---

### 2.8.9 Removing a Package

The package was removed using:

```bash
sudo apt remove tree
```

APT reported that one package would be removed and approximately **132 kB** of disk space would be freed.

The removal completed successfully.

The package was then verified after clearing Bash's remembered command paths:

```bash
hash -r
command -v tree
ls -l /usr/bin/tree
dpkg -S /usr/bin/tree
dpkg -l tree
```

Verification results:

```text
command -v tree
```

produced no output.

```text
ls: cannot access '/usr/bin/tree': No such file or directory
```

The package database reported:

```text
dpkg-query: no path found matching pattern /usr/bin/tree
```

and:

```text
dpkg-query: no packages found matching tree
```

This confirms that the package and its executable were successfully removed.

The earlier appearance of `/usr/bin/tree` immediately after removal was caused by Bash retaining the previously resolved executable path. Running:

```bash
hash -r
```

cleared the cached command location and allowed the removal to be verified correctly.

---

### 2.8.10 Inspecting APT Package Information

APT package priorities and repository configuration were inspected using:

```bash
apt-cache policy
```

The output showed the configured Debian repositories with their package priorities, including:

- `trixie`
- `trixie-updates`
- `trixie-security`

The default repository priority was generally `500`, while the installed package database had priority `100`.

No manually pinned packages were configured.

Package database statistics were also inspected using:

```bash
apt-cache stats
```

The command displayed statistics regarding package names, versions, descriptions, dependencies, virtual packages, and repository metadata.

The local APT package database contained approximately **160,762 package names** and accounted for approximately **46.4 MB** of package metadata.

---

### 2.8.11 Package Management Concepts Demonstrated

The practical exercises demonstrated the following package-management workflow:

```text
Repository Configuration
        ↓
apt update
        ↓
Package Search
        ↓
Package Information
        ↓
Package Installation
        ↓
Package Verification
        ↓
Package Status / Version Inspection
        ↓
Package Removal
        ↓
Removal Verification
```

The exercise also demonstrated the relationship between the two major Debian package-management layers:

- **APT** manages repositories, package dependencies, installation, upgrades, and removal.
- **dpkg** manages installed Debian packages and maintains the local package database.

---

### Result

Package management on the Debian server was successfully tested using `apt` and `dpkg`.

The following tasks were successfully completed:

- Identified `apt`, `apt-get`, and `dpkg`
- Inspected Debian repository configuration
- Updated APT package metadata
- Identified available package upgrades
- Searched for a package
- Inspected package metadata
- Installed the `tree` package
- Verified its executable, package status, version, and installed files
- Used the installed package
- Inspected APT package priorities and statistics
- Removed the package
- Verified that the package and executable were completely removed

---

## 2.9 Environment Variables

### Objective

The objective of this section was to understand environment variables and shell variables in Bash, including their scope, inheritance, modification, and role in command execution.

The following areas were examined:

- Inspecting existing environment variables
- Understanding commonly used shell environment variables
- Comparing environment variables with shell variables
- Exporting variables to child processes
- Understanding variable scope
- Inspecting and modifying the `PATH`
- Verifying command locations
- Inspecting Bash configuration files
- Verifying the active shell and process

---

### 2.9.1 Inspecting Environment Variables

The current environment was inspected using:

```bash
printenv
```

The environment contained variables such as:

```text
SHELL=/bin/bash
HOME=/home/alan
USER=alan
LOGNAME=alan
PWD=/home/alan
LANG=en_IN
LANGUAGE=en_IN:en
TERM=xterm-256color
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
SSH_TTY=/dev/pts/0
SSH_CLIENT=192.168.71.1 62741 22
SSH_CONNECTION=192.168.71.1 62741 192.168.71.136 22
XDG_SESSION_TYPE=tty
XDG_SESSION_ID=3
XDG_RUNTIME_DIR=/run/user/1000
```

The output confirms that the current shell is operating as user `alan` with `/home/alan` as the home directory and `/bin/bash` as the shell. Pasted text

---

### 2.9.2 Inspecting Common Environment Variables

Individual environment variables were examined using:

```bash
echo "$HOME"
echo "$USER"
echo "$SHELL"
echo "$PATH"
echo "$PWD"
echo "$OLDPWD"
echo "$LANG"
echo "$TERM"
```

The results included:

```text
HOME=/home/alan
USER=alan
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
PWD=/home/alan
LANG=en_IN
TERM=xterm-256color
```

`OLDPWD` was currently unset in the displayed shell output.

These variables provide information about the current user's environment, shell, working directory, language, terminal type, and executable search path. Pasted text

---

### 2.9.3 Environment Variables and Shell Variables

The environment was inspected using:

```bash
env | sort | head -30
```

Shell variables and Bash-specific variables were inspected using:

```bash
set | head -30
```

The `set` output contained Bash-specific variables such as:

```text
BASH=/bin/bash
BASH_VERSION='5.2.37(1)-release'
HOME=/home/alan
HOSTNAME=DEB-SERVER
HOSTTYPE=x86_64
HISTSIZE=1000
HISTFILESIZE=2000
```

The exported environment was inspected using:

```bash
export | head -20
```

Variables displayed with `declare -x` are exported variables available to child processes. Pasted text

---

### 2.9.4 Testing a Shell Variable

A shell variable was created without exporting it:

```bash
LAB_NAME="Linux Server Lab"
```

The value was available in the current shell:

```bash
echo "$LAB_NAME"
```

Output:

```text
Linux Server Lab
```

However, when a new child Bash process was started:

```bash
bash -c 'echo "$LAB_NAME"'
```

the variable was not available to the child process.

This demonstrates that a normal shell variable belongs to the current shell environment and is not automatically inherited by child processes. Pasted text

---

### 2.9.5 Exporting an Environment Variable

The variable was exported using:

```bash
export LAB_NAME="Linux Server Lab"
```

The value was then available both in the current shell and in a child shell:

```bash
echo "$LAB_NAME"
bash -c 'echo "$LAB_NAME"'
printenv LAB_NAME
```

The child shell returned:

```text
Linux Server Lab
```

and `printenv` also displayed:

```text
Linux Server Lab
```

This demonstrates that exported variables are inherited by child processes. Pasted text

---

### 2.9.6 Inspecting the PATH Variable

The current `PATH` was examined using:

```bash
echo "$PATH"
```

The initial value was:

```text
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

Individual directories were displayed using:

```bash
printf '%s\n' "$PATH" | tr ':' '\n'
```

Result:

```text
/usr/local/bin
/usr/bin
/bin
/usr/local/games
/usr/games
```

The `PATH` variable defines the directories that the shell searches when a command is executed without specifying its complete path. Pasted text

---

### 2.9.7 Verifying Command Locations

The locations of common commands were checked using:

```bash
command -v bash
command -v ls
command -v sudo
command -v apt
```

The results included:

```text
/usr/bin/bash
alias ls='ls --color=auto'
/usr/bin/sudo
/usr/bin/apt
```

The `ls` result showed that it is configured as a Bash alias, while the other commands resolved to executable paths. Pasted text

---

### 2.9.8 Temporarily Modifying PATH

A personal executable directory was created:

```bash
mkdir -p "$HOME/bin"
```

The directory was then added to the beginning of the current shell's `PATH`:

```bash
export PATH="$HOME/bin:$PATH"
```

The resulting value was:

```text
/home/alan/bin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

The directories were verified using:

```bash
printf '%s\n' "$PATH" | tr ':' '\n' | head
```

The new `$HOME/bin` directory appeared first in the search path. Pasted text

This modification affected the current shell session and was not permanently written to `.bashrc` or `.profile`.

---

### 2.9.9 Comparing Variable Scope

Two variables were created:

```bash
TEMP_VAR="temporary"
export EXPORTED_VAR="exported"
```

A child Bash process was then started:

```bash
bash -c 'echo "TEMP_VAR=$TEMP_VAR"; echo "EXPORTED_VAR=$EXPORTED_VAR"'
```

The result was:

```text
TEMP_VAR=
EXPORTED_VAR=exported
```

This demonstrates the difference between the two variables:

| Variable       | Exported | Available to child process |
| -------------- | -------- | -------------------------- |
| `TEMP_VAR`     | No       | No                         |
| `EXPORTED_VAR` | Yes      | Yes                        |

Therefore, exporting a variable places it into the environment inherited by child processes. Pasted text

---

### 2.9.10 Inspecting Shell Configuration Files

The home directory was inspected using:

```bash
ls -la ~
```

The Bash configuration files were present:

```text
/home/alan/.bashrc
/home/alan/.profile
```

Their permissions and ownership were:

```text
-rw-r--r-- 1 alan alan 3560 ... /home/alan/.bashrc
-rw-r--r-- 1 alan alan  807 ... /home/alan/.profile
```

The files were specifically inspected using:

```bash
ls -la ~/.bashrc ~/.profile 2>/dev/null
```

Pasted text

Configuration assignments were searched using:

```bash
grep -nE '^(export|PATH=|[A-Za-z_][A-Za-z0-9_]*=)' ~/.bashrc ~/.profile 2>/dev/null
```

The matching assignments found in `.bashrc` included:

```text
HISTCONTROL=ignoreboth
HISTSIZE=1000
HISTFILESIZE=2000
```

No explicit `PATH=` or `export` assignment was returned by this search in either file. Pasted text

---

### 2.9.11 Verifying the Current Shell

The active shell was checked using:

```bash
echo "$SHELL"
```

Output:

```text
/bin/bash
```

The current shell process was inspected using:

```bash
ps -p $$ -o pid,ppid,user,stat,cmd
```

Output:

```text
PID    PPID USER  STAT CMD
1277   1276 alan  Ss   -bash
```

This confirms that the current session is running under Bash as user `alan`. Pasted text

---

### Result

Environment variables and shell variables were successfully examined and tested on the Debian server.

The following concepts were demonstrated:

- Inspecting the current environment with `printenv`
- Inspecting shell variables with `set`
- Identifying exported variables with `export`
- Creating ordinary shell variables
- Exporting variables to child processes
- Demonstrating variable inheritance
- Inspecting and modifying `PATH`
- Verifying command locations using `command -v`
- Creating a personal `$HOME/bin` directory
- Inspecting `.bashrc` and `.profile`
- Verifying the active Bash shell and its process

The practical tests demonstrated that **shell variables remain local to the current shell unless exported**, while **exported variables are inherited by child processes**.

---

## 2.10 Shell Configuration

### Objective

The objective of this section was to understand and configure the Bash shell environment, including shell startup files, aliases, command history, shell options, functions, and persistent user-specific configuration.

The following areas were examined:

- Identifying the active shell
- Inspecting Bash startup files
- Understanding `.profile` and `.bashrc`
- Inspecting system-wide Bash configuration
- Examining Bash shell options
- Inspecting aliases and functions
- Understanding command history configuration
- Creating temporary aliases
- Testing `.bashrc` reload behavior
- Creating persistent environment variables
- Creating persistent aliases
- Verifying the resulting shell configuration

---

### 2.10.1 Identifying the Active Shell

The active shell was identified using:

```bash id="1"
echo "$SHELL"
ps -p $$ -o pid,ppid,user,stat,cmd
echo "$0"
```

The results showed:

```text id="2"
SHELL=/bin/bash
PID=1301
PPID=1300
USER=alan
STAT=Ss
CMD=-bash
```

The active shell is therefore **Bash**, running as user `alan`. The `-bash` process indicates the current Bash login shell. Pasted text

---

### 2.10.2 Bash Startup Files

The user-specific Bash startup files were inspected using:

```bash id="3"
ls -la ~/.bashrc ~/.profile ~/.bash_logout 2>/dev/null
```

The following files were present:

```text id="4"
~/.bashrc
~/.profile
~/.bash_logout
```

They were owned by `alan` and were readable by the user. Pasted text

The `.profile` file contains configuration for login shells. It checks whether Bash is running and sources `.bashrc` when available:

```bash id="5"
if [ -n "$BASH_VERSION" ]; then
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
```

It also adds `$HOME/bin` and `$HOME/.local/bin` to `PATH` when those directories exist. Pasted text

---

### 2.10.3 Inspecting `.bashrc`

The `.bashrc` file was inspected to understand the configuration applied to interactive Bash sessions.

The file contains settings for:

- Interactive-shell detection
- Command history
- Bash shell options
- Terminal window size
- Prompt configuration
- `ls` color support
- Aliases
- Bash completion

The configuration prevents `.bashrc` from executing its interactive configuration when the shell is not interactive:

```bash
case $- in
    *i*) ;;
      *) return;;
esac
```

Pasted text

---

### 2.10.4 Command History Configuration

The Bash history configuration was inspected using:

```bash id="6"
echo "HISTSIZE=$HISTSIZE"
echo "HISTFILESIZE=$HISTFILESIZE"
echo "HISTCONTROL=$HISTCONTROL"
echo "HISTFILE=$HISTFILE"
```

The configuration was:

```text id="7"
HISTSIZE=1000
HISTFILESIZE=2000
HISTCONTROL=ignoreboth
HISTFILE=/home/alan/.bash_history
```

The `.bashrc` configuration also enables:

```bash
shopt -s histappend
```

This causes new history entries to be appended to the history file instead of replacing existing entries. Pasted text

The history file was verified:

```bash id="8"
ls -l "$HISTFILE"
```

It was owned by `alan` with permissions:

```text id="9"
-rw------- 1 alan alan ... /home/alan/.bash_history
```

The history configuration was also confirmed directly in `.bashrc`. Pasted text

---

### 2.10.5 Inspecting System-Wide Bash Configuration

The system-wide configuration files were inspected:

```bash id="10"
ls -la /etc/profile /etc/bash.bashrc 2>/dev/null
```

Both files were present:

```text id="11"
/etc/profile
/etc/bash.bashrc
```

Relevant configuration was inspected using:

```bash id="12"
grep -nE '^[[:space:]]*(export|alias|function|[A-Za-z_][A-Za-z0-9_]*=)' /etc/profile /etc/bash.bashrc 2>/dev/null | head -50
```

The system-wide configuration includes the system `PATH`, exports it, configures the shell prompt, and defines the `command_not_found_handle` function. Pasted text

This demonstrates the distinction between:

- **System-wide configuration** — `/etc/profile`, `/etc/bash.bashrc`
- **User-specific configuration** — `~/.profile`, `~/.bashrc`

---

### 2.10.6 Inspecting Bash Shell Options

The active Bash options were inspected using:

```bash id="13"
set -o
```

Important enabled options included:

```text id="14"
braceexpand    on
emacs          on
hashall        on
histexpand     on
history        on
interactive-comments on
monitor        on
```

Options such as `errexit`, `noclobber`, `noglob`, `nounset`, `pipefail`, and `xtrace` were disabled in the current shell.

The `posix` option was also disabled, confirming that the shell was operating with normal Bash behavior rather than POSIX mode. Pasted text

---

### 2.10.7 Inspecting Aliases and Functions

Current aliases were inspected using:

```bash id="15"
alias
```

The configured aliases included:

```text id="16"
alias cls='clear'
alias ls='ls --color=auto'
```

The command type of several commands was checked:

```bash id="17"
type ls
type cd
type pwd
type echo
```

The results demonstrated that:

- `ls` is an alias
- `cd` is a Bash shell builtin
- `pwd` is a Bash shell builtin
- `echo` is a Bash shell builtin

Pasted text Pasted text

A large number of Bash completion-related functions were also present, loaded by the Bash completion configuration. Pasted text

---

### 2.10.8 Testing a Temporary Alias

A temporary alias was created:

```bash id="18"
alias ll='ls -lah'
```

It was verified using:

```bash id="19"
alias ll
ll
```

The alias successfully expanded to `ls -lah` and displayed the contents of the user's home directory.

A child Bash process was then started:

```bash id="20"
bash -c 'type ll'
```

The child shell returned:

```text id="21"
bash: line 1: type: ll: not found
```

This demonstrates that an interactively created alias is local to the current shell and is not automatically inherited by child shells. Pasted text

---

### 2.10.9 Creating a Persistent Environment Variable

A test environment variable was added to `.bashrc`:

```bash id="22"
echo 'export SHELL_CONFIG_TEST="enabled"' >> ~/.bashrc
```

The configuration was reloaded without opening a new shell:

```bash id="23"
source ~/.bashrc
```

The variable was then verified:

```bash id="24"
echo "$SHELL_CONFIG_TEST"
printenv SHELL_CONFIG_TEST
```

Both commands returned:

```text id="25"
enabled
```

The variable was also available to a child Bash process:

```bash id="26"
bash -c 'echo "$SHELL_CONFIG_TEST"'
```

which returned:

```text id="27"
enabled
```

The configuration entry was confirmed in `.bashrc`:

```text id="28"
export SHELL_CONFIG_TEST="enabled"
```

Pasted text

This demonstrates that configuration placed in `.bashrc` can be loaded into the current shell with `source` and will be applied to subsequently started interactive Bash sessions.

---

### 2.10.10 Creating a Persistent Alias

A practical `ll` alias was added to `.bashrc`:

```bash id="29"
echo "alias ll='ls -lah'" >> ~/.bashrc
```

The configuration was reloaded:

```bash id="30"
source ~/.bashrc
```

The alias was verified:

```bash id="31"
alias ll
ll
```

The alias was successfully loaded and executed:

```text id="32"
alias ll='ls -lah'
```

The `ll` command then displayed the contents of the home directory in long, human-readable format. Pasted text

---

### 2.10.11 Verifying the Final Configuration

The final portion of `.bashrc` was inspected using:

```bash id="33"
tail -20 ~/.bashrc
```

The resulting configuration contained:

```text id="34"
# User Aliases
alias cls="clear"
export SHELL_CONFIG_TEST="enabled"
alias ll='ls -lah'
```

The configuration was verified in the current shell:

```bash id="35"
echo "$SHELL_CONFIG_TEST"
alias ll
```

The results confirmed:

```text id="36"
SHELL_CONFIG_TEST=enabled
alias ll='ls -lah'
```

The current shell process remained Bash running as user `alan`. Pasted text

---

### Result

Shell configuration on the Debian server was successfully examined and modified.

The following tasks were completed:

- Identified Bash as the active shell
- Inspected `.profile`, `.bashrc`, and `.bash_logout`
- Inspected system-wide Bash configuration
- Examined Bash shell options
- Inspected command history configuration
- Examined aliases and shell functions
- Demonstrated the scope of temporary aliases
- Created and tested a persistent environment variable
- Created and tested a persistent `ll` alias
- Reloaded `.bashrc` using `source`
- Verified the final Bash configuration

The practical exercise demonstrated how **user-specific Bash configuration can be stored in `~/.bashrc` and reloaded into an interactive shell**, while system-wide configuration is maintained separately under `/etc`.

---



## Phase Status

**Status: Completed**