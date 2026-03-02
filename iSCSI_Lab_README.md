# iSCSI Storage Lab – Complete Implementation Guide

## Overview

This project demonstrates a complete setup of an iSCSI (Internet Small Computer System Interface) storage environment on Ubuntu Linux.

The objective of this lab was to understand how remote storage can behave like a local disk. Instead of only reading documentation, the entire configuration was implemented and tested step by step.

This setup simulates a small Storage Area Network (SAN) where a remote disk appears as a local block device.

---

## What is iSCSI?

iSCSI is a block-level storage protocol that transmits SCSI commands over a TCP/IP network.

Unlike file-based storage systems such as NFS or SMB:

- File-based storage → Access files  
- iSCSI → Access raw disk blocks  

With iSCSI, the remote storage appears as:

- `/dev/sdb`
- A physical hard disk
- A disk that can be formatted and mounted normally

This makes iSCSI widely used in virtualization platforms, enterprise storage systems, and cloud block storage environments.

---

## Architecture Overview

### Logical Architecture

Target → TCP/IP Network → Initiator → Local Block Device

### Main Components

| Component | Description |
|------------|-------------|
| Target | Storage server providing disk |
| Initiator | Client system connecting to storage |
| Backstore | Physical storage (file/disk/LVM) |
| LUN | Logical disk exposed to client |
| IQN | Unique iSCSI identifier |

---

## Lab Setup Architecture

In this lab, both Target and Initiator run on the same Ubuntu VM using `127.0.0.1`.

Flow:

Backstore (disk1.img)  
→ Target (targetcli)  
→ TCP/IP (127.0.0.1)  
→ Initiator (open-iscsi)  
→ `/dev/sdb`

---

## Lab Environment

| Component | Value |
|------------|--------|
| Operating System | Ubuntu |
| Setup Type | Single Virtual Machine |
| Portal IP | 127.0.0.1 |

---

## Installation

Update the system and install required packages:

```bash
sudo apt update
sudo apt install targetcli-fb open-iscsi -y
```

- `targetcli-fb` → Used to configure iSCSI target  
- `open-iscsi` → Used as iSCSI initiator  

---

## Target Configuration

Enter target configuration shell:

```bash
sudo targetcli
```

### Step 1 – Create Backstore

```bash
/backstores/fileio create disk1 /root/disk1.img 2G
```

This creates a 2GB file that behaves like a disk.

---

### Step 2 – Create Target

```bash
/iscsi create iqn.2026-03.local.lab:target1
```

IQN (iSCSI Qualified Name) uniquely identifies the target.

---

### Step 3 – Create LUN

```bash
/iscsi/iqn.2026-03.local.lab:target1/tpg1/luns create /backstores/fileio/disk1
```

This exposes the backstore as a logical disk.

---

### Step 4 – Disable Authentication (Lab Only)

```bash
set attribute authentication=0
```

Note: Authentication (CHAP) should be enabled in production environments.

---

### Step 5 – Create ACL

```bash
acls create iqn.2026-03.local.lab:client1
```

This allows the initiator with matching IQN to connect.

---

## Initiator Configuration

Edit initiator name:

```bash
sudo nano /etc/iscsi/initiatorname.iscsi
```

Make sure it matches:

```
iqn.2026-03.local.lab:client1
```

Restart service:

```bash
sudo systemctl restart open-iscsi
```

---

## Discover Target

```bash
sudo iscsiadm -m discovery -t sendtargets -p 127.0.0.1
```

If configuration is correct, the target IQN will appear.

---

## Login to Target

```bash
sudo iscsiadm -m node --login
```

The remote disk will now connect to the system.

---

## Verify Disk

```bash
lsblk
```

Expected output:

```
sdb   2G
```

The remote storage is now visible as `/dev/sdb`.

---

## Perform I/O Operations

Format the disk:

```bash
sudo mkfs.ext4 /dev/sdb
```

Create mount directory:

```bash
sudo mkdir /mnt/iscsi
```

Mount disk:

```bash
sudo mount /dev/sdb /mnt/iscsi
```

Now files created inside `/mnt/iscsi` are stored on the iSCSI disk.

---

## Real-World Applications

iSCSI is widely used in:

- Enterprise data centers  
- Virtualization platforms (VMware, Proxmox, etc.)  
- Cloud block storage systems  
- Database storage  
- Backup and disaster recovery  

---

## Key Learnings

- Difference between block storage and file storage  
- How SAN architecture works  
- How remote storage appears as a local disk  
- Practical configuration of Target and Initiator  
- Importance of authentication and ACL  

---

## Future Improvements

- Enable CHAP authentication  
- Separate Target and Initiator into different machines  
- Integrate with Kubernetes  
- Use LVM-based backstore  
- Configure automatic login at boot  

---

## References

IETF RFC 3720  
https://datatracker.ietf.org/doc/html/rfc3720  

Open-iSCSI GitHub  
https://github.com/open-iscsi/open-iscsi  

Red Hat Storage Documentation  
https://docs.redhat.com  

DigitalOcean – Block Storage Guide  
https://www.digitalocean.com/resources/articles/block-storage  

---
