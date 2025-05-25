---
title: Google Cloud Create Local SSD Disks for VMs
description: Learn to Create Local SSD Disks for VMs on Google Cloud Platform GCP
---

## Step-01: Introduction
- What is a Local SSD Disk ?
- Implement a demo with Local SSD Disk
- Persistent Disk vs Local SSD

## Step-02: Create new VM Instance
- Verify [OS COMPATABILITY FOR BEST PERFORMANCE](https://cloud.google.com/compute/docs/disks/local-ssd#choose_an_interface)
- Go to Compute Engine -> VM Instances -> **CREATE INSTANCE**
- **Name:** demo9-vm-localssd
- **Region:** us-central
- **Zone:** us-central1-a
- **Machine family:** GENERAL-PURPOSE
- **Series:** N2  (NOT ALL Machine SERIES SUPPORT Local SSDs)
- **Machine type:** n2-standard-2 
- **Boot Disk** 
    - **Operating System:** Debian
    - **Version:** Debian/GNU Linux 9 (Stretch) - VERIFY OS COMPATABILITY MATRIX 
    - **Boot disk type:** Balanced persistent disk
    - **Size:** 10 GB    
- REST ALL LEAVE TO DEFAULTS
- Click on **ADVANCED OPTIONS**
- Clcik on **Disks**
- Click on **ADD LOCAL SSD**
- Finally click  on **CREATE** to create VM Instance
```t
# Create VM Instance with local-ssd disk
gcloud compute instances create demo9-vm-localssd \
    --project=gcplearn9 \
    --zone=us-central1-a \
    --machine-type=n2-standard-2 \
    --network-interface=subnet=default \
    --local-ssd-recovery-timeout=1 \
    --tags=http-server \
    --local-ssd=interface=NVME 
```

## Step-03: Mount local SSD Block Device in VM Instance
```t
# Verify local SSD disk in VM Instance Properties 
Goto -> Compute Engine -> VM Instances -> Clcik on "demo9-vm-localssd"
Verify "Local disks" 

# Connect to VM instance using cloud shell 
gcloud compute ssh --zone "us-central1-a" "demo9-vm-localssd" --project "gcplearn9"

# Display information about Block Devices
sudo lsblk

# Format local ssd with an ext4 file system
sudo mkfs.ext4 -F /dev/[SSD_NAME]
sudo mkfs.ext4 -F /dev/nvme0n1

# Create Directory to mount the local ssd device
sudo mkdir -p /mnt/disks/disk1ssd

# Mount local ssd on the VM instance
sudo mount /dev/[SSD_NAME] /mnt/disks/[MNT_DIR]
sudo mount /dev/nvme0n1 /mnt/disks/disk1ssd

# Enable Read and Write access to the device
sudo chmod a+w /mnt/disks/[MNT_DIR]
sudo chmod a+w /mnt/disks/disk1ssd

# Verify File system
sudo df -h

##### SAMPLE OUTPUT ######
dkalyanreddy@demo9-vm-localssd:~$ sudo df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3.9G     0  3.9G   0% /dev
tmpfs           796M  448K  796M   1% /run
/dev/sda1       9.7G  2.0G  7.2G  22% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/sda15      124M   12M  113M  10% /boot/efi
tmpfs           796M     0  796M   0% /run/user/1000
/dev/nvme0n1    369G   28K  350G   1% /mnt/disks/disk1ssd
dkalyanreddy@demo9-vm-localssd:~$ 
```

## Step-04: Automatically Mount the Block Device when Instance Reboots
```t
# Connect to VM instance using cloud shell 
gcloud compute ssh --zone "us-central1-a" "demo9-vm-localssd" --project "gcplearn9"

# Automatically mount the Block Device when Instance Reboots
## Backup /etc/fstab
sudo cp /etc/fstab /etc/fstab_backup_before_localssd

## List UUID of Block Device
sudo blkid -s UUID
Take UUID of "/dev/nvme0n1:" and replace in below command

## Update the /etc/fstab
echo UUID=fd6b5842-ac95-4a93-a215-0a24967f606b /mnt/disks/disk1ssd ext4 discard,defaults,nofail 0 2 | sudo tee -a /etc/fstab

## Verify /etc/fstab
cat /etc/fstab
```

## Step-05: Verify Local SSD
```t
# Create a file in local ssd
sudo echo "Welcome to Stacksimplify - VM Hostname: $(hostname) VM IP Address: $(hostname -I)" | sudo tee /mnt/disks/disk1ssd/localssd.html

# Verify File
cat /mnt/disks/disk1ssd/localssd.html
```

## Step-06: Verify Local SSD after system reboot
```t
# Verify Uptime before reboot
uptime

# Run Command
sudo reboot

# Connect to VM instance using cloud shell 
gcloud compute ssh --zone "us-central1-a" "demo9-vm-localssd" --project "gcplearn9"

# Verify VM Uptime after reboot
uptime

# Verify Disk
df -h
ls -lrt /mnt/disks/disk1ssd/
cat /mnt/disks/disk1ssd/localssd.html
exit
```

## Step-07: Stop or Delete VM to avoid charges
```t
# Stop VM Instance
gcloud compute instances stop demo9-vm-localssd --zone=us-central1-a
gcloud compute instances list --filter='name:demo9-vm-localssd'

Note-1: If a VM has any attached Local SSD disks, you must use the --discard-local-ssd flag to indicate whether or not the Local SSD data should be discarded. 
Note-2: TO PRESERVE LOCAL SSD: To stop the VM and preserve the Local SSD data when
you stop the VM by setting --discard-local-ssd=False.
Note-3: TO DISCARD LOCAL SSD: To stop the VM and discard the Local SSD data, specify     --discard-local-ssd=True.
Note-4: Preserving the Local SSD disk data incurs costs and is subject to limitations

# Delete VM 
gcloud compute instances delete demo9-vm-localssd --zone=us-central1-a
gcloud compute instances list --filter='name:demo9-vm-localssd'
```

## Step-08: Persistent Disk vs Local SSD
- Understand more about Persistent Disk vs Local SSD
- Refer presentation slides for the same.
---


# 🚀 Google Cloud Compute Engine: Local SSD Overview

Local SSDs in Google Cloud provide **high-performance, low-latency storage** physically attached to the **host server** of your VM instance. They are ephemeral, optimized for temporary and high-throughput use cases.

---

## 📌 What is a Local SSD?

- **Physically attached** to the **host server** where the VM resides.
- Offers **very high IOPS** and **low latency**, significantly outperforming persistent disks.
- **Data is lost** when the VM is stopped, deleted, or terminated (ephemeral).
- Ideal for **temporary storage needs** such as caching or scratch data.

---

## 🔐 Encryption Support

| Feature | Supported on Persistent Disks | Supported on Local SSD |
|--------|-------------------------------|------------------------|
| Google Managed Encryption | ✅ | ✅ (automatic) |
| Customer Managed Key (CMEK) | ✅ | ❌ |
| Customer Supplied Key (CSEK) | ✅ | ❌ |

- **Local SSDs are automatically encrypted** by Compute Engine.
- **CMEK/CSEK not supported** due to direct hardware-level integration.

---

## 🖥️ Machine Type Compatibility

- **Only specific machine types support local SSDs.**
- Use GCP documentation to verify support for **NVMe or SCSI (Multi-Queue)** interfaces:
  - [Local SSD Compatibility](https://cloud.google.com/compute/docs/disks/local-ssd#machine_types)

---

## ⚙️ Interface & Performance

| Interface | Notes |
|-----------|-------|
| **NVMe** | Recommended for modern Linux kernels; best performance |
| **SCSI (Multi-Queue)** | Use for legacy or specific kernel compatibility |

- **10x to 100x** performance increase over persistent disks.
- Best used with **NVMe-enabled images** or **multi-queue SCSI** images.

---

## 🔄 Lifecycle & Behavior

| Feature | Persistent Disk | Local SSD |
|---------|------------------|-----------|
| Lifecycle | Independent of VM | Tied to VM lifecycle |
| Snapshots | ✅ Supported | ❌ Not supported |
| Attach/Detach | ✅ Yes | ❌ No – one-time binding |
| Reuse across VMs | ✅ Yes | ❌ No |

---

## 🛡️ Durability & Availability

- **Persistent Disk**: High durability, availability, and flexibility.
- **Local SSD**: Lower durability and availability.
  - **Data is not persistent** and is lost on VM stop/delete.
  - **Enable live migration** to preserve data across host maintenance.

---

## 📚 Use Cases

| Use Case | Why Local SSD? |
|----------|----------------|
| Caching | Extremely fast read/write for temporary cache data |
| Temporary high-speed buffer | ETL pipelines, temp scratch space |
| Data analytics | Low-latency intermediate storage for processing |

---

## 📊 Comparison Summary

| Feature | Persistent Disk | Local SSD |
|--------|------------------|-----------|
| Storage Type | Permanent (durable) | Ephemeral (temporary) |
| Encryption | Google/CMEK/CSEK | Automatic only |
| Machine Types | All supported | Limited |
| Performance | Good | 10–100x faster |
| Snapshot Support | ✅ Yes | ❌ No |
| Detach/Attach | ✅ Yes | ❌ No |
| Mount Type | Network drive | Physical host-attached |
| Use Case | General purpose | Temporary high-performance data |

---

## 🛠️ Next Steps

➡ Proceed to the **implementation demo**, where we:
- Create a VM with local SSD
- Format and mount the SSD
- Measure performance
- Observe data lifecycle on VM termination

---

## 📖 References

- [GCP Local SSD Documentation](https://cloud.google.com/compute/docs/disks/local-ssd)
- [Machine Type Support](https://cloud.google.com/compute/docs/disks/local-ssd#machine_types)
- [NVMe & SCSI Performance Notes](https://cloud.google.com/compute/docs/disks/performance#local_ssd_performance)

```

