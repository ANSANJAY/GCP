---
title: Google Cloud Create Non Boot Disk and attach to VM
description: Learn to Create Non Boot Disk and attach to VM on Google Cloud Platform GCP
---

## Step-01: Introduction
1. Create Non-Boot disk using Compute Engine Storage Disks
2. Attach the non-boot Disk to VM
3. Mount the disk in VM
4. Create some files and verify the same 

## Step-02: Create a new VM Instance
```t
# Create VM Instance
gcloud compute instances create demo7-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --network-interface=subnet=default \
  --tags=http-server \
  --metadata-from-file=startup-script=webserver-install.sh 

# List Compute Instances
gcloud compute instances list   
```

## Step-03: Create Disk (Non-Boot)
- Go to Compute Engine -> Storage -> Disks -> Create Disk
- **Name:** mydisk1
- **Description:** mydisk1
- **Location:** Single Zone
- **Region:** us-central1
- **Zone:** us-central1-a
- **Disk source type:** Blank disk
- **Disk settings:** 
  - **Disk type:** Balanced Persistent Disk
  - **Click on COMPARE DISK TYPES** - UNDERSTAND ABOUT VARIOUS DISK OPTIONS
  - **Size:** 15GB
- **Snapshot schedule (Recommended):**  leave empty (leave to defaults)
- **Encryption:** 
  - **Customer Managed Encryption Key (CMEK):** kalyankey1    
  - Click on **GRANT** to provide access to `The service-562846405594@compute-system.iam.gserviceaccount.com service account does not have the "cloudkms.cryptoKeyEncrypterDecrypter" role. Verify the service account has permission to encrypt/decrypt with the selected key.`
- **ADD LABEL:** environment=dev 
- Click on **CREATE**
```t
# Createa a Blank Disk
gcloud compute disks create mydisk1 \
    --project=gcplearn9 \
    --type=pd-balanced \
    --description=mydisk1 \
    --size=15GB \
    --zone=us-central1-a

# Createa a Blank Disk with KMS Key
gcloud compute disks create mydisk1 \
    --project=gcplearn9 \
    --type=pd-balanced \
    --description=mydisk1 \
    --size=15GB \
    --zone=us-central1-a \
    --kms-key=projects/gcplearn9/locations/global/keyRings/my-keyring3/cryptoKeys/kalyankey1    
```

## Step-04: Attach this disk to VM
- Go to Compute Engine -> VM Instances -> demo1-vm -> Edit
- **Additional Disks** 
  - Click on **Attach existing disk**
  - **Disk:** mydisk1
  - **Mode:** Read/Write
  - **Deletion Rule:** keep disk
  - **Device Name:** Based on disk name (default)
  - Click on **SAVE**
- Click on **SAVE**

## Step-05: List disks that are attached to your instance
- `sdb` is the device name for the new blank persistent disk.
```t
# Connect to VM instance using cloud shell
gcloud compute ssh --zone "us-central1-a" "demo7-vm" --project "gcplearn9"

# Use the symlink created for your attached disk to determine which device to format.
ls -l /dev/disk/by-id/google-*

## Sample Output
dkalyanreddy@demo1-vm:~$ ls -l /dev/disk/by-id/google-*
lrwxrwxrwx 1 root root  9 Mar 27 09:16 /dev/disk/by-id/google-demo1-vm -> ../../sda
lrwxrwxrwx 1 root root 10 Mar 27 09:16 /dev/disk/by-id/google-demo1-vm-part1 -> ../../sda1
lrwxrwxrwx 1 root root 11 Mar 27 09:16 /dev/disk/by-id/google-demo1-vm-part14 -> ../../sda14
lrwxrwxrwx 1 root root 11 Mar 27 09:16 /dev/disk/by-id/google-demo1-vm-part15 -> ../../sda15
lrwxrwxrwx 1 root root  9 Mar 27 09:16 /dev/disk/by-id/google-disk1-nonboot-app1 -> ../../sdb
dkalyanreddy@demo1-vm:~$

# Format disk using mkfs command
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/DEVICE_NAME
sudo mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb

# Create Mount Directory
sudo mkdir -p /mnt/disks/MOUNT_DIR
sudo mkdir -p /mnt/disks/myapp1

# Mount Disk
sudo mount -o discard,defaults /dev/DEVICE_NAME /mnt/disks/MOUNT_DIR
sudo mount -o discard,defaults /dev/sdb /mnt/disks/myapp1

# Enable Read Write permissions on disk
sudo chmod a+w /mnt/disks/MOUNT_DIR
sudo chmod a+w /mnt/disks/myapp1

# Verify Mount Point
df -h

## Sample Output
dkalyanreddy@demo7-vm:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            478M     0  478M   0% /dev
tmpfs            98M  388K   98M   1% /run
/dev/sda1       9.7G  2.0G  7.2G  22% /
tmpfs           488M     0  488M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda15      124M   11M  114M   9% /boot/efi
tmpfs            98M     0   98M   0% /run/user/1000
/dev/sdb         15G   24K   15G   1% /mnt/disks/myapp1


# Create File in newly mouted disk
echo "New disk attached" >> /mnt/disks/myapp1/newdisk.txt
cat /mnt/disks/app1-disk/newdisk.txt
```

## Step-06: Configure automatic mounting on VM restart
```t
# Backup fstab
sudo cp /etc/fstab /etc/fstab.backup

# Use the blkid command to list the UUID for the disk.
sudo blkid /dev/DEVICE_NAME
sudo blkid /dev/sdb

# Create Mountpoint entry and update in /etc/fstab
UUID=UUID_VALUE /mnt/disks/MOUNT_DIR ext4 discard,defaults,MOUNT_OPTION 0 2
UUID=88dfae9c-9c25-4e99-81a6-17825a5bd70b /mnt/disks/myapp1 ext4 discard,defaults,nofail 0 2

# Make the mount point permanent by adding in /etc/fstab (if not after VM reboot it will not be available)
sudo vi /etc/fstab
UUID=88dfae9c-9c25-4e99-81a6-17825a5bd70b /mnt/disks/myapp1 ext4 discard,defaults,nofail 0 2
[or]
echo UUID=88dfae9c-9c25-4e99-81a6-17825a5bd70b /mnt/disks/myapp1 ext4 discard,defaults,nofail 0 2 | sudo tee -a /etc/fstab

# Verify /etc/fstab
cat /etc/fstab

# Run mount -a to see if any errors
sudo mount -a
exit
```
## Step-07: Stop and Start VM to verify if new disk still attached to VM
```t
## Reboot VM
# Stop VM Instance
gcloud compute instances stop demo7-vm --zone=us-central1-a
gcloud compute instances list --filter='name:demo7-vm'

# Start VM Instance
gcloud compute instances start demo7-vm --zone=us-central1-a
gcloud compute instances list --filter='name:demo7-vm'

# Connect to VM instance using cloud shell and Verify the new disk attached
gcloud compute ssh --zone "us-central1-a" "demo7-vm" --project "gcplearn9"
df -h
cat /mnt/disks/myapp1/newdisk.txt
echo "my new file 101" >> /mnt/disks/myapp1/mynewfile.txt
ls /mnt/disks/myapp1/
cat /mnt/disks/myapp1/mynewfile.txt
exit
```

## Step-08: Clean-Up to Avoid Charges
- DONT DELETE THESE, WE WILL USE ion DEMO 02-03 later, just STOP THE VM
```t
# Stop VM Instance
gcloud compute instances stop demo7-vm --zone=us-central1-a
gcloud compute instances list --filter='name:demo7-vm'

# Delete VM 
gcloud compute instances delete demo7-vm --zone=us-central1-a
gcloud compute instances list --filter='name:demo7-vm'

# Delete Disk 
gcloud compute disks list 
gcloud compute disks delete mydisk1 --zone=us-central1-a
gcloud compute disks list 
```

## Additional Reference
- [Add Persistent Disk](https://cloud.google.com/compute/docs/disks/add-persistent-disk)
- [Format and Mount Disk](https://cloud.google.com/compute/docs/disks/format-mount-disk-linux)

---
Here's the full **README format** documentation based on the persistent disk demo and lecture content:

````md
---
## Title: Google Cloud Persistent Disks (Zonal & Regional)
## Description: Learn the types, performance, lifecycle, and creation methods for Persistent Disks in GCP Compute Engine.
---

## ✅ Step-by-Step Summary

### 🔹 Types of Persistent Disks

| Type                  | Description                                  | Scope         | Use Case                          |
|-----------------------|----------------------------------------------|---------------|-----------------------------------|
| Zonal Persistent Disk | Replicated in a **single zone**              | Zonal         | General workloads                 |
| Regional Persistent Disk | Replicated in **two zones** in same region | Multi-Zonal   | High availability VMs             |

- Persistent Disks are **network-attached block storage**
- They can be used as both **boot** and **data** disks (except regional disks which are data-only)
- **Lifecycle is independent** of VM — can be detached/reattached or retained after VM deletion
- Disks can be **resized (increased)** even when attached or live

---

## 💡 Key Differences: Zonal vs Regional Persistent Disk

| Feature                        | Zonal PD        | Regional PD                |
|-------------------------------|-----------------|----------------------------|
| Replication                   | Single zone     | Two zones in one region    |
| Boot Disk Support             | ✅ Yes          | ❌ No                       |
| Min Size                      | 10 GB           | 200 GB                     |
| VM Compatibility              | All types       | E2, N1, N2, N2D only       |
| From Snapshot Support         | ✅ Yes          | ✅ Yes                      |
| From Image Support            | ✅ Yes          | ❌ No                       |

---

## 🔁 Real-World Use (with Your Project Experience)

- Used **regional persistent disks** in failover-ready VM clusters for enhanced resilience.
- Resized disks live during performance spikes without downtime.
- Created VM templates using PD-balanced boot disks for cost-effective general-purpose compute.
- Used snapshots to clone production environments for testing safely and quickly.

---

## 📊 Performance Metrics Summary

- **Performance is proportional to**: `Disk Size` + `vCPU Count of the VM`
- Metrics affected: `Read IOPS`, `Write IOPS`, `Read Throughput`, `Write Throughput`

| Disk Size | vCPU | Read IOPS | Write IOPS | Read MB/s | Write MB/s |
|-----------|------|-----------|------------|------------|-------------|
| 500 GB    | 2    | ~375      | ~750       | ~60        | ~60         |
| 1000 GB   | 4    | Higher     | Higher     | Higher     | Higher      |

- You **cannot shrink** disk size, only increase.
- Performance scales **automatically**, unlike Hyperdisks which allow explicit tuning.

---

## 💽 Disk Type Comparison

| Disk Type         | Code         | Cost     | Durability | Boot Support | Performance      | Best For                          |
|-------------------|--------------|----------|------------|--------------|------------------|-----------------------------------|
| Standard (HDD)    | `pd-standard`| Low      | 99.29%     | ✅ Yes*      | 🟥 Slow           | Cold data, backups, low cost use  |
| Balanced (SSD)    | `pd-balanced`| Moderate | 99.999%    | ✅ Default   | 🟨 Good           | General workloads (default)       |
| SSD               | `pd-ssd`     | High     | 99.999%    | ✅           | 🟩 Very Good      | Latency-sensitive apps, DBs       |
| Extreme SSD       | `pd-extreme` | Very High| 99.9999%   | ✅*          | 🟩🟩 Excellent     | SAP HANA, HPC, write-intensive    |
| Hyperdisks        | `hyperdisk-*`| Custom   | NA         | ❌           | Custom-defined   | Next-gen workloads (next demo)    |

> \* Bootable only when explicitly configured or allowed.

---

## 🔧 CLI Command Summary

```bash
# Create Zonal Persistent Disk
gcloud compute disks create my-zonal-pd \
  --size=100GB \
  --type=pd-balanced \
  --zone=us-central1-a

# Create Regional Persistent Disk
gcloud compute disks create my-regional-pd \
  --size=200GB \
  --type=pd-ssd \
  --region=us-central1 \
  --replica-zones=us-central1-a,us-central1-b

# Resize a Persistent Disk
gcloud compute disks resize my-zonal-pd \
  --size=200GB \
  --zone=us-central1-a

# List Persistent Disks
gcloud compute disks list

# Delete a Disk
gcloud compute disks delete my-zonal-pd \
  --zone=us-central1-a
````

---

## 🎯 Interview Questions + Sample Answers

**Q1: What's the difference between zonal and regional persistent disks?**
**A:** Zonal PD is stored in a single zone. Regional PD replicates data across two zones for higher availability but cannot be used as a boot disk.

---

**Q2: Can you resize a persistent disk while it’s in use?**
**A:** Yes, persistent disks can be resized at any time—while attached or detached—and data remains intact.

---

**Q3: Why can’t you decrease the size of a persistent disk?**
**A:** Reducing size can lead to data corruption. GCP only supports size increases to avoid loss of existing data.

---

**Q4: Which disk type would you use for SAP HANA?**
**A:** `pd-extreme` is recommended due to its high durability, IOPS, and throughput requirements.

---

## 🧠 Analogy to Remember

💽 **Persistent Disks = Cloud SSDs/HDDs**

* **Zonal PD**: Like keeping your files on one local drive.
* **Regional PD**: Like using RAID-1 across two separate offices.
* Resize = Add more storage blocks to your SSD.
* Cannot shrink = Once you give more space, it's yours forever.

---

## 📚 Additional References

* [Persistent Disks Overview](https://cloud.google.com/compute/docs/disks)
* [Regional PDs](https://cloud.google.com/compute/docs/disks#regionalpd)
* [Performance Guide](https://cloud.google.com/compute/docs/disks/performance)
* [Hyperdisks Overview](https://cloud.google.com/compute/docs/disks/hyperdisk-overview)

```



