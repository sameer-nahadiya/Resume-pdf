# iSCSI Storage Lab -- Complete Implementation Guide

## 📘 Overview

This project demonstrates the complete setup and configuration of an
iSCSI (Internet Small Computer System Interface) storage environment on
Ubuntu Linux. The lab simulates a SAN (Storage Area Network) where a
remote disk appears as a local block device.

------------------------------------------------------------------------

## 🔹 What is iSCSI?

iSCSI is a block-level storage protocol that transmits SCSI commands
over TCP/IP networks. Unlike file-based protocols (NFS/SMB), iSCSI
presents storage as a raw disk device.

------------------------------------------------------------------------

## 🏗 Architecture Components

-   **Target** -- Storage Server
-   **Initiator** -- Client system
-   **Backstore** -- Actual storage (file/disk/LVM)
-   **LUN** -- Logical Unit Number (virtual disk)
-   **IQN** -- Unique identifier

------------------------------------------------------------------------

## 🖥 Lab Environment

  Component   Value
  ----------- -----------
  OS          Ubuntu
  Setup       Single VM
  Portal IP   127.0.0.1

------------------------------------------------------------------------

## ⚙ Installation

``` bash
sudo apt update
sudo apt install targetcli-fb open-iscsi -y
```

------------------------------------------------------------------------

## 🧱 Target Configuration

### Create Backstore

``` bash
sudo targetcli
/backstores/fileio create disk1 /root/disk1.img 2G
```

### Create Target

``` bash
/iscsi create iqn.2026-03.local.lab:target1
```

### Create LUN

``` bash
/iscsi/iqn.2026-03.local.lab:target1/tpg1/luns create /backstores/fileio/disk1
```

### Disable Authentication (Lab Only)

``` bash
set attribute authentication=0
```

### Create ACL

``` bash
acls create iqn.2026-03.local.lab:client1
```

------------------------------------------------------------------------

## 🔌 Initiator Configuration

Edit:

``` bash
sudo nano /etc/iscsi/initiatorname.iscsi
```

Restart:

``` bash
sudo systemctl restart open-iscsi
```

------------------------------------------------------------------------

## 🔍 Discover Target

``` bash
sudo iscsiadm -m discovery -t sendtargets -p 127.0.0.1
```

------------------------------------------------------------------------

## 🔐 Login

``` bash
sudo iscsiadm -m node --login
```

------------------------------------------------------------------------

## 💾 Verify Disk

``` bash
lsblk
```

You should see:

    sdb  2G

------------------------------------------------------------------------

## 📂 Perform I/O

``` bash
sudo mkfs.ext4 /dev/sdb
sudo mkdir /mnt/iscsi
sudo mount /dev/sdb /mnt/iscsi
```

------------------------------------------------------------------------

## 🚀 Real-World Applications

-   Data Centers
-   Virtualization Platforms
-   Cloud Block Storage
-   Enterprise Databases

------------------------------------------------------------------------

## 📚 References

-   IETF RFC 3720 
-   Open-iSCSI GitHub --> (https://github.com/open-iscsi/open-iscsi.git)
-   Red Hat Blog -- iSCSI Guide --> (https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/managing_storage_devices/configuring-an-iscsi-initiator_managing-storage-devices)
-   DigitalOcean -- Block Storage Explained --> (https://www.digitalocean.com/resources/articles/block-storage)

------------------------------------------------------------------------

