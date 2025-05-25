---
title: Google Cloud Create VM instance with Disk Snapshot
description: Learn to Create VM instance with Disk Snapshot on Google Cloud Platform GCP
---

## Step-01: Introduction
- Understand Compute Engine -> Storage -> Disks
- VM Boot Disk
  - Public Images
  - Custom Images
  - Snapshots
  - Existing Disks
- Create a Disk Snapshot
- Create VM Instance with Disk Snapshot

## Step-02: Start VM demo4-vm-from-machine-image
- Go to Compute Engine -> VM Instances -> demo8-vm1 -> Actions -> Start

## Step-03: Create Snapshot
- **Option-1:** Go to Compute Engine -> Storage -> Disks -> demo8-vm1 -> Actions -> Create Snapshot
- **Option-2:** Go to Compute Engine -> Storage -> Snapshots -> Create Snapshot
- **Name:** demo8-vm1-snapshot-1
- **Description:** demo8-vm1-snapshot-1
- **source disk:** demo8-vm1
- **Location:** Mutli-regional
- **Select Location:** us (United States)
- **Encryption:** Uses same encryption as disk
- **Application consistency:** unchecked (leave to defaults)
- Click on **Create**
```t
# Create Snapshot
gcloud compute snapshots create demo8-vm1-snapshot \
    --project=gcplearn9 \
    --description=demo8-vm1-snapshot \
    --source-disk=demo8-vm1 \
    --source-disk-zone=us-central1-a \
    --storage-location=us
```

## Step-04: Review Snapshot Properties
- Go to Compute Engine -> Storage -> Snapshots ->  demo8-vm1-snapshot
- Review Snapshot Properties

## Step-05: Create VM or Disk using Snapshot
- Go to Compute Engine -> Storage -> Snapshots ->  demo8-vm1-snapshot
- Review **CREATE INSTANCE**
- Review **CREATE DISK**

## Step-06: Discuss about Creating Snapshot Schedule
1. **Schedule Options:** Hourly, Daily, Weekly
2. **Autodeletion snapshots after:** 14
3. **Deletion Rule:** Keep Snapshots or Delete Snapshots older than 14 days


## Additional Reference
- [Linux application consistent persistent disk snapshot](https://cloud.google.com/compute/docs/disks/creating-linux-application-consistent-pd-snapshots)

---

# 📸 Persistent Disk Snapshots in Google Cloud

This guide provides an in-depth understanding of **Persistent Disk Snapshots** in Google Cloud Platform (GCP), their use cases, types, scheduling options, and best practices.

---

## ✅ Step-by-Step Summary

1. Snapshots are **backups** of zonal or regional persistent disks.
2. Snapshots can be taken **while the VM is running** (no downtime needed).
3. Snapshots are **global resources** and can be **shared across projects**.
4. Snapshots are **incremental by default**, saving space and costs.
5. You can define **snapshot schedules** (hourly/daily/weekly).
6. Use **deletion rules** to manage old snapshots and control cost.

---

## 💡 Real-World Use (with Project Application)

- Take daily backups of production VM disks to protect against data loss.
- Use snapshot schedules to automate backups of critical databases.
- Share a snapshot across environments (e.g., staging → prod) to maintain data consistency.

---

## 💾 Why Are Snapshots Important?

- Provide **point-in-time backups** of VM disks
- Help restore quickly from **accidental deletion** or **corruption**
- Used to **migrate** disks or replicate environments

---

## 🖥️ Can You Take Snapshots While VM Is Running?

**Yes.** Snapshots can be taken without stopping or detaching the disk:
> "Snapshots can be created from disks even while they are attached to running VMs."

This makes them ideal for continuous workloads that require minimal downtime.

---

## 🌍 Snapshot Scope

- Snapshots are **global resources**
- Can be restored:
  - In **same region**
  - In **different regions**
  - Across **different projects** (if shared)

---

## 🕒 Snapshot Frequency & Scheduling

### ⏱️ Scheduling Options:
- **Hourly**
- **Daily**
- **Weekly**

### ⚠️ Best Practices:
- **Avoid snapshots during business hours** to minimize performance degradation
- **1–2 snapshots per day** is sufficient for most workloads
- Schedule backups during **non-peak hours**

---

## 🔄 Incremental Snapshots

### How It Works:
- First snapshot is **full**
- Subsequent snapshots only save **differences**
- **No data loss** occurs if older snapshots are deleted

### Advantages:
- Lower storage cost
- Faster snapshot creation
- Efficient use of network bandwidth

Google automatically captures a **full snapshot periodically** to maintain history reliability.

---

## 🧹 Snapshot Deletion Rules

Set rules to delete older snapshots:
- Example: `Delete snapshots older than 14 days`
- Helps avoid unnecessary storage charges

This can be configured in **Snapshot Schedules**.

---

## 🚀 Recommended Workflow

If you're **frequently creating disks from a snapshot**, it's better to:

1. **Convert snapshot → custom image**
2. **Use the image** to create VMs

Benefits:
- Faster VM creation
- Reduced networking cost

---

## 🛠️ Interview Questions + Sample Answers

**Q1: Can we take a snapshot from a running VM?**  
**A:** Yes, snapshots can be taken even when the disk is attached to a running VM.

---

**Q2: Are GCP snapshots full or incremental?**  
**A:** Snapshots are incremental. Only changed data blocks are saved after the initial full snapshot.

---

**Q3: What's the best practice for snapshot frequency?**  
**A:** Once or twice per day, scheduled during non-business hours to avoid performance impact.

---

**Q4: Can you delete older snapshots?**  
**A:** Yes, deletion rules in snapshot schedules help manage storage cost and retain only recent backups.

---

## 📚 References

- [Persistent Disk Snapshots Documentation](https://cloud.google.com/compute/docs/disks/snapshots)
- [Snapshot Schedules](https://cloud.google.com/compute/docs/disks/scheduled-snapshots)


