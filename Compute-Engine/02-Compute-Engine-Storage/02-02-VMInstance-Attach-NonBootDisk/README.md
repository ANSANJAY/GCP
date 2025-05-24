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
## Title: Google Cloud Persistent Disks (Zonal & Regional)

## Description: Learn the types, performance, lifecycle, and creation methods for Persistent Disks in GCP Compute Engine.


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

Here's the complete **README format** for your Compute Engine Persistent Disk concepts demo:

````md
---
title: Google Cloud Compute Engine Persistent Disks
description: Learn about zonal and regional persistent disks, disk types, performance behavior, and use cases in GCP Compute Engine.
---

## ✅ Step-by-Step Summary

- Persistent Disks (PDs) are **network-attached block storage** for Compute Engine VMs.
- They can be used as **boot or data disks** (with some limitations).
- PDs come in two main scopes:
  - **Zonal PD** – Replicated in one zone.
  - **Regional PD** – Replicated in two zones (same region) for high availability.
- Disk performance **scales automatically** with disk size and the **vCPU count** of the VM.
- PDs are **detachable**, **resizable (only increase)**, and **survive VM deletion** (if configured).
- Supported disk types: `pd-standard`, `pd-balanced`, `pd-ssd`, `pd-extreme`, and `hyperdisk-*` (limited boot support).

---

## 💡 Real-World Use (with Your Project Experience)

- Used **PD-balanced** boot disks for general-purpose application servers.
- Resized live data disks from 50 GB → 200 GB to support growing storage without downtime.
- Employed **regional persistent disks** in critical environments to ensure **zone-level failover** and recovery.
- Preferred **extreme PDs** in latency-sensitive batch processing pipelines for analytics workloads.

---

## 📊 Disk Type Comparison

| Disk Type      | Storage Tech | Bootable | Cost       | Durability     | Performance Level      | Best Use Case                          |
|----------------|--------------|----------|------------|----------------|------------------------|----------------------------------------|
| `pd-standard`  | HDD          | ✅\*     | Low        | 99.29%         | 🟥 Low                 | Cold data, backups                     |
| `pd-balanced`  | SSD          | ✅       | Moderate   | 99.999%        | 🟨 Good                | General-purpose workloads              |
| `pd-ssd`       | SSD          | ✅       | High       | 99.999%        | 🟩 Very Good           | Databases, latency-sensitive workloads |
| `pd-extreme`   | SSD          | ✅\*     | Very High  | 99.9999%       | 🟩🟩 Excellent          | SAP HANA, HPC                          |
| `hyperdisk-*`  | SSD (custom) | ❌\*     | Custom     | NA             | User-defined           | Granular throughput/I/O needs          |

> \* Bootable only in certain scenarios or not recommended.

---

## 🔁 Lifecycle & Attachment Options

- Disks can be:
  - Attached/detached at runtime
  - Used across VMs (ReadOnly)
  - Resized live (only expansion allowed)
- **Data persists** independently of VM lifecycle (except for **local SSDs**).
- Regional PDs support failover **across zones** but only on supported VM families.

---

## 🧪 Persistent Disk Creation Sources

| Source Type          | Supported     |
|----------------------|---------------|
| Blank disk           | ✅            |
| Snapshot             | ✅ (zonal + regional) |
| Image                | ✅ (only zonal) |
| Instance snapshot    | ✅            |
| Archive snapshot     | ✅            |

---

## 📈 Disk Performance = f(Disk Size, vCPU Count)

| Disk Size | vCPU | Read IOPS | Write IOPS | Read MB/s | Write MB/s |
|-----------|------|-----------|------------|------------|-------------|
| 500 GB    | 2    | ~375      | ~750       | ~60        | ~60         |
| 1000 GB   | 4    | Higher     | Higher     | Higher     | Higher      |

- Auto-scaling performance (unlike Hyperdisks where it's configurable)
- Higher disk size or vCPUs → better performance
- No performance tuning knobs on regular PDs

---

## 🧠 Analogy to Remember

💾 **Persistent Disk = USB Drive for VMs**
- **Zonal PD**: A reliable USB you plug into one computer.
- **Regional PD**: A synced USB drive shared across two computers in separate rooms.
- **Performance**: Like USB 2.0 vs 3.0 — more size or CPU = faster speed.
- **Lifecycle**: You can unplug and reuse without losing files.

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

# Resize a Disk (increase only)
gcloud compute disks resize my-zonal-pd \
  --size=200GB \
  --zone=us-central1-a

# List All Disks
gcloud compute disks list

# Delete a Disk
gcloud compute disks delete my-zonal-pd \
  --zone=us-central1-a
````

---

## 🎯 Interview Questions + Sample Answers

**Q1: What’s the key difference between zonal and regional PDs?**
**A:** Zonal PDs are stored in one zone. Regional PDs are replicated across two zones for high availability and can survive zone outages.

---

**Q2: Can you shrink a persistent disk in GCP?**
**A:** No, resizing is one-way. You can only increase disk size to avoid data corruption risk.

---

**Q3: When would you use extreme SSDs over standard ones?**
**A:** For ultra-low latency and high-throughput workloads like SAP HANA, HPC, or analytics pipelines with intensive read/write needs.

---

**Q4: What happens if you delete a VM with a persistent disk?**
**A:** If the deletion protection flag isn’t set, the disk remains unless you explicitly choose to delete it with the VM.

---

## 📚 Additional References

* [Persistent Disk Overview](https://cloud.google.com/compute/docs/disks)
* [Regional Persistent Disks](https://cloud.google.com/compute/docs/disks#regionalpd)
* [Performance Guide](https://cloud.google.com/compute/docs/disks/performance)
* [Hyperdisk Overview](https://cloud.google.com/compute/docs/disks/hyperdisk-overview)

```
---
## title: Google Cloud Compute Engine - Persistent Disk (Data Disk) Attachment
## description: Learn how to create a zonal data disk, encrypt it with CMEK, attach it to a VM, and prepare it for use.
---

## ✅ Step-by-Step Summary

1. **Create a new VM instance** using `gcloud compute instances create`.
2. **Create a new zonal persistent disk** (`my-disk-1`) with 15 GB capacity.
3. Use **Balanced PD** (`pd-balanced`) and **Customer-Managed Encryption Key (CMEK)** for encryption.
4. **Grant required KMS permissions** to the Compute Engine default service account.
5. **Attach the disk** to the created VM as a **read-write** device.
6. In the next step (not covered here), you will **mount and format** the disk on the VM instance.

---

## 💡 Real-World Use (with Your Project Experience)

- Created encrypted data disks using CMEK for sensitive workloads.
- Ensured data disks were retained even if VMs were deleted, by disabling auto-deletion.
- Configured KMS roles and permissions on service accounts to enable secure encryption key usage.
- Performed disk mounts and used them to store logs, application data, or backups.

---

## 🧠 Analogy to Remember

📀 **Attaching a Data Disk = Plugging in an External Hard Drive**
- Until you **mount** it in your OS, it’s just a plugged-in device.
- Using CMEK is like locking the drive with your **own digital key** instead of letting the vendor manage it.
- You can unplug it (detach) anytime and reuse it elsewhere.

---

## 🔧 CLI Command Summary

### 1. Create VM
```bash
gcloud compute instances create demo7-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro
````

### 2. Create Data Disk with CMEK

```bash
gcloud compute disks create my-disk-1 \
  --size=15GB \
  --type=pd-balanced \
  --zone=us-central1-a \
  --kms-key=projects/<PROJECT_ID>/locations/us-central1/keyRings/my-key-ring-1/cryptoKeys/my-symmetric-key-1
```

### 3. Attach Disk to VM

```bash
gcloud compute instances attach-disk demo7-vm \
  --disk=my-disk-1 \
  --zone=us-central1-a \
  --device-name=my-disk-1 \
  --mode=rw \
  --no-delete-disks
```

> Make sure your service account (typically `PROJECT_NUMBER-compute@developer.gserviceaccount.com`) has the following IAM role on the key:
> `roles/cloudkms.cryptoKeyEncrypterDecrypter`

---

## 🧪 Web Console Steps

1. **Compute Engine > VM Instances > Create Instance**
2. Create disk: `Compute Engine > Disks > Create Disk`

   * Name: `my-disk-1`
   * Type: `Balanced Persistent Disk`
   * Size: `15 GB`
   * Encryption: `Customer-managed key (CMEK)`
   * Key: `my-symmetric-key-1` from `my-key-ring-1`
3. **Attach Disk to VM**:

   * Edit `demo7-vm`
   * Go to Additional Disks > Attach existing disk
   * Mode: `Read/Write`
   * Deletion rule: `Keep disk`
   * Save

---

## 🧩 What’s Next?

> In the next step (separate README), you’ll SSH into the VM and:
>
> * List available disks
> * Format and mount `my-disk-1`
> * Create files to verify functionality

---

## 🎯 Interview Questions + Sample Answers

**Q1: What’s the difference between boot disk and data disk in GCP?**
**A:** Boot disk contains the OS and is required to start the VM. Data disk is an additional volume for storing application data, logs, or backups. Data disks can be attached/detached independently of the VM lifecycle.

---

**Q2: How does CMEK differ from Google-managed encryption?**
**A:** CMEK allows customers to manage their own encryption keys via Cloud KMS, offering more control and compliance, while Google-managed keys are automatic and invisible to the user.

---

**Q3: Can you attach a persistent disk to multiple VMs?**
**A:** Yes, if it is attached in **read-only mode** to multiple VMs. For **read-write multi-writer**, special configurations are needed and not supported in all use cases.

---

**Q4: What permissions are needed to use CMEK on a disk?**
**A:** The VM's service account must have the `roles/cloudkms.cryptoKeyEncrypterDecrypter` role on the KMS key.

---

## 📚 Additional References

* [Persistent Disks Overview](https://cloud.google.com/compute/docs/disks)
* [Using CMEK with Compute Engine](https://cloud.google.com/compute/docs/disks/customer-managed-encryption)
* [Attach a Disk to VM](https://cloud.google.com/compute/docs/disks/add-persistent-disk)
* [gcloud disk commands](https://cloud.google.com/sdk/gcloud/reference/compute/disks/)

```
Here’s the **README** documenting the entire disk mounting and persistence configuration in Google Cloud:

````md
---
title: Mount and Persist a Data Disk on Google Cloud Compute Engine
description: Learn how to mount a data disk, write files, configure fstab for auto-mount, and verify persistence after VM reboot.
---

## ✅ Step-by-Step Summary

1. Connect to the VM instance via SSH.
2. Identify the attached data disk using device ID.
3. Format the disk using `ext4`.
4. Create a mount directory (`/mnt/disks/my-app-1`).
5. Mount the disk to the directory.
6. Set read/write permissions.
7. Create a test file to validate write access.
8. Use `UUID` and `/etc/fstab` to ensure the disk auto-mounts after reboot.
9. Reboot the VM and verify mount and data persistence.

---

## 💡 Real-World Use (with Your Project Experience)

- Successfully formatted and mounted customer-managed encrypted disks in secure environments.
- Ensured persistence of data disks after reboots using `fstab` and UUID-based entries.
- Used mounted disks for storing application logs and runtime files separate from OS volume.
- Automated provisioning via `gcloud` and mounted using shell scripts in startup configurations.

---

## 🧠 Analogy to Remember

🗂️ **Mounting a Disk = Labeling a Drawer**
- You plugged in a new disk (external drawer).
- Until you label and open it (`mount`), it’s just there, unused.
- Writing a file inside is like putting documents into the drawer.
- `fstab` is your sticky note: “Always reopen this drawer when restarting.”

---

## 🔧 CLI Command Summary

### 1. SSH into the VM
```bash
gcloud compute ssh demo7-vm --zone=us-central1-a
````

### 2. List Disks

```bash
ls -l /dev/disk/by-id/google-*
```

### 3. Format the Disk (ext4)

```bash
sudo mkfs.ext4 -F /dev/sdb
```

### 4. Create Mount Point

```bash
sudo mkdir -p /mnt/disks/my-app-1
```

### 5. Mount the Disk

```bash
sudo mount /dev/sdb /mnt/disks/my-app-1
```

### 6. Set Permissions

```bash
sudo chmod a+w /mnt/disks/my-app-1
```

### 7. Verify Disk Mount

```bash
df -h
```

### 8. Create a File on the Disk

```bash
cd /mnt/disks/my-app-1
echo "New disk attached" | sudo tee new-disk.txt
```

---

## 🧩 Auto-Mount on Reboot Using fstab

### 1. Get Disk UUID

```bash
sudo blkid /dev/sdb
```

Example UUID:

```
UUID="1a2b3c4d-5678-90ef-gh12-34567890abcd"
```

### 2. Backup fstab and Edit

```bash
sudo cp /etc/fstab /etc/fstab.bak
sudo vi /etc/fstab
```

Add the following line:

```
UUID=1a2b3c4d-5678-90ef-gh12-34567890abcd /mnt/disks/my-app-1 ext4 defaults 0 2
```

### 3. Validate fstab Entry

```bash
sudo mount -a
```

---

## 🔁 Reboot and Verify

### 1. Reboot VM

```bash
gcloud compute instances stop demo7-vm --zone=us-central1-a
gcloud compute instances start demo7-vm --zone=us-central1-a
```

### 2. SSH and Check Mount

```bash
gcloud compute ssh demo7-vm --zone=us-central1-a
df -h
cat /mnt/disks/my-app-1/new-disk.txt
```

---

## 🎯 Interview Questions + Sample Answers

**Q1: What happens to an attached disk if a VM is deleted?**
**A:** Unless configured to auto-delete, the disk persists and can be reattached to another VM.

---

**Q2: Why use UUIDs in `/etc/fstab` instead of device names like `/dev/sdb`?**
**A:** Device names can change across reboots; UUIDs are stable and prevent mount failures.

---

**Q3: Can you mount a Google persistent disk on multiple VMs?**
**A:** Only in **read-only mode** (or special **multi-writer** mode); read-write mount by multiple VMs is restricted by default.

---

**Q4: What is the role of `fstab` in VM disk management?**
**A:** It defines static mounts to be automatically activated at boot, based on disk UUIDs or device paths.

---

## 📚 Additional References

* [Mounting Disks on Linux VMs](https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting)
* [fstab Manual](https://man7.org/linux/man-pages/man5/fstab.5.html)
* [Cloud KMS Integration with Disks](https://cloud.google.com/compute/docs/disks/customer-managed-encryption)

```

````md
---
## Title: Resize Boot and Data Disks in Google Cloud Compute Engine
## Description: Learn how to resize boot and non-boot (data) persistent disks, including automatic and manual resizing steps.
---

## ✅ Step-by-Step Summary

1. Stop the VM before resizing disks.
2. Resize the **boot disk** in the GCP Console or via `gcloud`.
3. Resize the **data disk** (non-boot) the same way.
4. Start the VM and SSH into it.
5. **Boot disk** will auto-expand (for public images like Debian/Ubuntu).
6. **Data disk** requires manual resizing using `resize2fs`.
7. Validate new disk sizes using `df -h`.
8. Optionally delete VM and verify the data disk remains.
9. Clean up by deleting the data disk manually.

---

## 💡 Real-World Use (with Your Project Experience)

- Successfully resized encrypted persistent disks without affecting data.
- Automated boot disk expansion in CI workloads.
- Manually resized data disks to accommodate log growth in high-throughput applications.
- Verified lifecycle independence of data disks by reusing them across VMs.

---

## 🧠 Analogy to Remember

📦 **Resizing Disks = Expanding a Box**
- Boot disk: GCP auto-expands the box for you.
- Data disk: You get a bigger box but must arrange the space inside manually (`resize2fs`).

---

## 🔧 CLI Command Summary

### 1. Resize Boot Disk via Console (Demo7 VM)

- Go to **Compute Engine → Disks → demo7-vm**
- Click **Edit**, change size from `10GB` → `20GB`, click **Save**

### 2. Resize Data Disk (my-disk-1)

- Go to **Disks → my-disk-1**
- Click **Edit**, change size from `15GB` → `25GB`, click **Save**

### 3. Start the VM

```bash
gcloud compute instances start demo7-vm --zone=us-central1-a
````

### 4. SSH into the VM

```bash
gcloud compute ssh demo7-vm --zone=us-central1-a
```

---

## 🧪 Verify Resize Results

### 5. View Disk Usage

```bash
df -h
```

You’ll see:

* `/` (boot disk) → Auto-expanded to 20GB
* `/mnt/disks/my-app-1` → Still shows 15GB (manual step needed)

---

## 🛠️ Manually Resize Data Disk (Non-Boot Disk)

### 6. Run Resize Command

```bash
sudo resize2fs /dev/sdb
```

### 7. Confirm Resize

```bash
df -h | grep sdb
```

You should see:

```
/dev/sdb      25G   0G   ...  /mnt/disks/my-app-1
```

---

## 🗑️ Delete VM & Validate Disk Lifecycle

### 8. Delete the VM

```bash
gcloud compute instances delete demo7-vm --zone=us-central1-a
```

### 9. Check If Data Disk Still Exists

```bash
gcloud compute disks list
```

✅ `my-disk-1` should still be present — it is **not deleted** with the VM.

---

## 🧹 Final Cleanup (Delete Data Disk)

```bash
gcloud compute disks delete my-disk-1 --zone=us-central1-a
```

---

## 🎯 Interview Questions + Sample Answers

**Q1: Can a persistent disk be resized without VM downtime?**
**A:** Yes, but resizing the **disk** itself doesn’t require downtime; however, applying the changes to a **boot disk** requires a **VM reboot**, and a **data disk** requires manual file system resizing.

---

**Q2: Does increasing a disk's size delete data?**
**A:** No. Disk resizing is non-destructive. However, shrinking a disk is **not allowed** in GCP.

---

**Q3: What’s the difference in boot disk vs data disk resizing behavior?**
**A:** Boot disks with public OS images auto-expand the root partition. Data disks require a manual `resize2fs` operation.

---

**Q4: What happens to data disks when VM is deleted?**
**A:** Unless set to auto-delete, **data disks persist** and can be reattached to other VMs.

---

## 📚 Additional References

* [Resizing Persistent Disks](https://cloud.google.com/compute/docs/disks#resize_pd)
* [Resize Boot Disks](https://cloud.google.com/compute/docs/disks/resize-persistent-disk#resize_boot)
* [Manual File System Expansion](https://cloud.google.com/compute/docs/disks/add-persistent-disk#formatting)

```
Here's a structured `README.md` for your **Regional Persistent Disk Demo** in Google Cloud Compute Engine:

````md
---
## Title: Regional Persistent Disk - GCP Compute Engine Demo
## Description: Step-by-step guide to creating and attaching a regional persistent disk in read-only mode to multiple VMs.
---

## ✅ Step-by-Step Summary

1. Create a **Regional Persistent Disk** using the GCP Console.
2. Attach this disk to **two different VMs** (VM1 and VM2) in **read-only mode**.
3. Attempt to attach the disk in **read-write** mode on the second VM — observe the limitation.
4. Clean up by deleting both VM instances and the disk using `gcloud`.

---

## 💡 Real-World Use (with Your Project Experience)

- Simulated a multi-zone GCP environment for **fault-tolerant static content delivery**.
- Demonstrated how **regional persistent disks** can be shared across zones in **read-only** mode.
- Highlighted safe practices for shared datasets in multi-VM deployment topologies.

---

## 📦 Analogy to Remember

🗂️ **Read-Only Shared Disk = Library Reference Book**  
You can have multiple readers (VMs), but nobody is allowed to write in the book. Perfect for content distribution or read-heavy apps.

---

## 🔧 CLI Command Summary

### 1. Create a Regional Persistent Disk (200GB)

```bash
gcloud compute disks create regional-disk-1 \
  --type=pd-balanced \
  --size=200GB \
  --region=us-central1 \
  --replica-zones=us-central1-a,us-central1-f
````

### 2. Create VM Instances

```bash
gcloud compute instances create vm1 --zone=us-central1-a
gcloud compute instances create vm2 --zone=us-central1-f
```

### 3. Attach Disk to VM1 (Read-Only)

* Go to `Compute Engine → VM Instances → vm1 → Edit`
* Scroll to **Disks** → Click **Attach existing disk**
* Choose `regional-disk-1`
* Set mode to `Read-only`
* Click **Save**

### 4. Attach Disk to VM2 (Read-Only)

* Same steps as above
* If you try `Read-write`, GCP will throw an error:

  > "Cannot set the disk to read-write mode because it is already attached to another instance."

---

## 🗑️ Clean-Up Commands

### 5. Delete VMs

```bash
gcloud compute instances delete vm1 vm2 --zone=us-central1-a --quiet
```

### 6. Delete the Regional Disk

```bash
gcloud compute disks delete regional-disk-1 \
  --region=us-central1 --quiet
```

---

## 🎯 Interview Questions + Sample Answers

**Q1: Why can't we attach a regional persistent disk in read-write mode to multiple VMs?**
**A:** GCP supports **read-only** multi-attachment for regional persistent disks. Read-write across VMs would risk **data corruption** unless explicitly managed, which is supported only with **multi-writer disks** under specific workloads.

---

**Q2: What are good use cases for regional persistent disks in read-only mode?**
**A:** Distributing **read-heavy static content**, shared data repositories across zones, **multi-zone high availability** scenarios where only read access is needed.

---

**Q3: Can you use a regional disk as a boot disk?**
**A:** No. Regional persistent disks **cannot be used as boot disks**. They are only usable as data disks.

---

## 📚 Additional References

* [GCP Regional Persistent Disks](https://cloud.google.com/compute/docs/disks#regionalpersistentdisks)
* [Attaching Disks in Read-Only Mode](https://cloud.google.com/compute/docs/disks/multiple-read-only)
* [Persistent Disk Types and Performance](https://cloud.google.com/compute/docs/disks/performance)

```








