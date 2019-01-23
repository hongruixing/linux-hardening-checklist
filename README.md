<p align="center">
    <img src="https://github.com/trimstray/working-template/blob/master/doc/img/main_preview.png"
        alt="Master">
</p>

<br>

<p align="center">
  <a href="https://github.com/trimstray/linux-hardening-checklist/tree/master">
    <img src="https://img.shields.io/badge/Branch-master-green.svg?longCache=true"
        alt="Branch">
  </a>
  <a href="https://github.com/trimstray/working-template/pulls">
    <img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?longCache=true"
        alt="Pull Requests">
  </a>
  <a href="http://www.gnu.org/licenses/">
    <img src="https://img.shields.io/badge/License-GNU-blue.svg?longCache=true"
        alt="License">
  </a>
</p>

<div align="center">
  <sub>Created by
  <a href="https://twitter.com/trimstray">trimstray</a> and
  <a href="https://github.com/trimstray/linux-hardening-checklist/graphs/contributors">
    contributors
  </a>
</div>

<br>

****

# Table of Contents

- **[Introduction](#introduction)**
  * [General disclaimer](#general-disclaimer)
  * [Levels of priority](#levels-of-priority)
  * [OpenSCAP](#openscap)
- **[Partitioning](#partitioning)**
  * [`/boot` partition](#boot-partition)
  * [`/home` partition](#home-partition)
  * [`/usr` partition](#usr-partition)
  * [`/var` partition](#var-partition)
  * [`/var/log` and `/var/log/audit` partitions](#varlog-and-varlogaudit-partitions)
  * [`/tmp` and `/var/tmp` partitions](#tmp-and-vartmp-partitions)
  * [`/dev/shm` shared memory](#devshm-shared-memory)
- **[Bootloader](#bootloader)**
- **[Linux Kernel](#linux-kernel)**
- **[Logging](#logging)**
- **[Users and Groups](#users-and-groups)**
- **[Permissions](#permissions)**
- **[SELinux & Auditd](#selinux--auditd)**
- **[System Updates](#system-updates)**
- **[Network](#network)**
- **[Services](#services)**
- **[Tools](#tools)**

# Introduction

  > In computing, **hardening** is usually the process of securing a system by reducing its surface of vulnerability, which is larger when a system performs more functions; in principle a single-function system is more secure than a multipurpose one. The main goal of systems hardening is to reduce security risk by eliminating potential attack vectors and condensing the system’s attack surface.

## General disclaimer

I'm not advocating throwing your existing hardening and deployment best practices out the door but I recommend is to always turn a feature from this checklist on in pre-production environments instead of jumping directly into production.

## Levels of priority

All items in this checklist contains three levels of priority:

* <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low"> means that the item has a **low** priority.
* <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium"> means that the item has a **medium** priority. You shouldn't avoid tackling that item.
* <img src="https://github.com/trimstray/working-template/blob/master/doc/img/high.png" alt="high"> means that the item has a **high** priority. You can't avoid following that rule and implement the corrections recommended.

## OpenSCAP

<img src="https://github.com/trimstray/working-template/blob/master/doc/img/openscap_logo.png" alt="OpenSCAP" align="left">

<p align="left"><b>SCAP</b> (<i>Security Content Automation Protocol</i>) provides a mechanism to check configurations, vulnerability management and evaluate policy compliance for a variety of systems. One of the most popular implementations of SCAP is <b>OpenSCAP</b> and it is very helpful for vulnerability assessment and also as hardening helper.

Some of the external audit tools use this standard. For example Nessus has functionality for authenticated SCAP scans.</p>

  > I tried to make this list fully compatible with OpenSCAP standard and rules.

# Partitioning

### `/boot` partition

- **Rule:** Ensure `/boot` located on separate partition. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > The idea behind the `/boot` partition was to make the partition always accessible to any machine that the drive was plugged into. As modern machines have lifted that restriction, there is no longer a fixed need for `/boot` to be separate, unless you require additional processing of the other partitions, such as encryption or file systems that are not natively recognized by the bootloader.

- **Rule:** Restrict `/boot` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium">

    **Rationale:**

    > The boot directory contains important files related to the Linux kernel, so you need to make sure that this directory is locked down to read-only permissions.

    **Example:**

    ```bash
    LABEL=/boot  /boot  ext2  defaults,nodev,nosuid,noexec,ro  1 2
    ```

### `/home` partition

- **Rule:** Ensure `/home` located on separate partition. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > Filling up the `/home` partition does not result in the main filesystem crashing or being unable to update. You can also re-install at any time or make data retrieval easier in the case of a crash. `/home` could be mounted to give users their home directories, with all their data files.

- **Rule:** Restrict `/home` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium">

    **Rationale:**

    > Users can create shared directories and execute scripts.

    **Example:**

    ```bash
    UUID=<...>  /home  ext4  defaults,nodev,nosuid  0 2
    ```

### `/usr` partition

- **Rule:** Ensure `/usr` located on separate partition. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > Some additional reasoning for isolating `/usr`, is for making it easier to deploy identical systems, these partitions can be prepared one time and then replicated across systems more easily.

- **Rule:** Restrict `/usr` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > It can be mounted read-only, offering a level of protection to the data under this directory so that it cannot be tampered with so easily.

    **Example:**

    ```bash
    UUID=<...>  /usr  ext4  defaults,nodev,ro  0 2
    ```

### `/var` partition

- **Rule:** Ensure `/var` located on separate partition. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/high.png" alt="high">

    **Rationale:**

    > `/var` can be filled up by user programs or daemons. Therefore it can be safe to have these in separate partitions that would prevent `/`, the root partition, to be 100% full, and would hit your system badly.

- **Rule:** Restrict `/var` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > This helps protect system services such as daemons or other programs which use it.

    **Example:**

    ```bash
    UUID=<...>  /var  ext4  defaults,nosuid  0 2
    ```

    **Comment:**

    > Some programs (like mail-mta/netqmail) will not be able to work properly if `/var` has `noexec` and `nosuid`. Consider removing those options if they cause problems.

### `/var/log` and `/var/log/audit` partitions

- **Rule:** Ensure `/var/log` and `/var/log/audit` located on separate partitions. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/high.png" alt="high">

    **Rationale:**

    > There are two important reasons to ensure that system logs are stored on a separate partition: protection against resource exhaustion (since logs can grow quite large) and protection of audit data.

- **Rule:** Restrict `/var` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > This helps protect system services such as daemons or other programs which use it.

    **Example:**

    ```bash
    UUID=<...>  /var/log        ext4  defaults,nosuid,noexec,nodev  0 2
    UUID=<...>  /var/log/audit  ext4  defaults,nosuid,noexec,nodev  0 2
    ```

### `/tmp` and `/var/tmp` partitions

- **Rule:** Ensure `/tmp` and `/var/tmp` located on separate partitions. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/high.png" alt="high">

    **Rationale:**

    > Several daemons/applications use the `/tmp` or `/var/tmp` directories to temporarily store data, log information, or to share information between their sub-components. However, due to the shared nature of these directories, several attacks are possible.

- **Rule:** Restrict `/var` and `/var/tmp` partitions mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium">

    **Rationale:**

    > Several daemons/applications use the `/tmp` or `/var/tmp` directories to temporarily store data, log information, or to share information between their sub-components.

    **Example:**

    ```bash
    mv /var/tmp /var/tmp.old
    ln -s /tmp /var/tmp
    cp -prf /var/tmp.old/* /tmp && rm -fr /var/tmp.old

    UUID=<...>  /tmp  ext4  defaults,nodev,nosuid,noexec  0 2
    ```

- **Rule:** Setting up polyinstantiated `/var` and `/var/tmp` directories. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium">

    **Rationale:**

    > Due to the shared nature of these directories, several attacks are possible. SELinux offers a solution in the form of polyinstantiated directories. This effectively means that both `/tmp/` and `/var/tmp/` are instantiated, making them appear private for each user.

    **Example:**

    ```bash
    # Create new directories:

    mkdir --mode 000 /tmp-inst
    mkdir --mode 000 /var/tmp/tmp-inst

    # Edit /etc/security/namespace.conf:

    /tmp      /tmp-inst/          level  root,adm
    /var/tmp  /var/tmp/tmp-inst/  level  root,adm

    # Set correct SELinux context:

    setsebool polyinstantiation_enabled=1
    chcon --reference=/tmp /tmp-inst
    chcon --reference=/var/tmp/ /var/tmp/tmp-inst
    ```

    **Comment:**

    > Don't do this for `/var/tmp` if this directory is mounted in `/tmp`.

### `/dev/shm` shared memory

- **Rule:** Restrict `/dev/shm` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/medium.png" alt="medium">

    **Rationale:**

    > One of the major security issue with the `/dev/shm` is anyone can upload and execute files inside the `/dev/shm` similar to the `/tmp` partition. Further the size should be limited to avoid an attacker filling up this mountpoint to the point where applications could be affected. (normally it allows 20% or more of RAM to be used). The sticky bit should be set like for any world writeable directory.

    **Example:**

    ```bash
    tmpfs  /dev/shm  tmpfs  rw,nodev,nosuid,noexec,size=1024M,mode=1777 0 0
    ```

- **Rule:** Set group for `/dev/shm`. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > You can also create a group named 'shm' and put application users for SHM-using applications in there.

    **Example:**

    ```bash
    tmpfs  /dev/shm  tmpfs  rw,nodev,nosuid,noexec,size=1024M,mode=1770,uid=root,gid=shm 0 0
    ```

### `/proc` filesystem

- **Rule:** Restrict `/prod` partition mount options. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > The proc pseudo-filesystem `/proc` should be mounted with hidepid. When setting hidepid to 2, directories entries in `/proc` will hidden.

    **Example:**

    ```bash
    proc  /proc  proc  defaults,hidepid=2  0 0
    ```

### `swap` partition

- **Rule:** Encrypted `swap` partition. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > The swap partition can hold a lot of unencrypted confidential information and the fact that it persists after shutting down the server can be a problem.

    **Example:**

    ```bash
    # Edit /etc/crypttab:
    sdb1_crypt /dev/sdb1 /dev/urandom cipher=aes-xts-plain64,size=256,swap,discard

    # Edit /etc/fstab:

    /dev/mapper/sdb1_crypt none swap sw 0 0
    ```

# Bootloader

### Protect bootloader config files

- **Rule:** Ensure bootloader config files are set properly permissions. <img src="https://github.com/trimstray/working-template/blob/master/doc/img/low.png" alt="low">

    **Rationale:**

    > Setting the permissions to read and write for root only prevents non-root users from seeing the boot parameters or changing them.

    **Example:**

    ```bash
    # Set the owner and group of /etc/grub.conf to the root user:
    chown root:root /etc/grub.conf
    chown -R root:root /etc/grub.d

    # Set permissions on the /etc/grub.conf or /etc/grub.d file to read and write for root only:
    chmod og-rwx /etc/grub.conf
    chmod -R og-rwx /etc/grub.d
    ```

# Linux Kernel

# Logging

# Users and Groups

# Permissions

# SELinux & Auditd

# System Updates

# Network

# Services

# Tools
