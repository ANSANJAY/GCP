---
title: Google Cloud Compute Engine Hyperdisks
description: Learn to Compute Engine Hyperdisks
---

## Step-01: Introduction
- Understand Hyperdisks
- Create Hyperdisk
 
## Step-02: Create Hyperdisk
- Go to Compute Engine -> Storage -> Disks
- **Name:** hyperdisk1
- **Description:** hyperdisk1
- **Location:** 
  - **Region:** us-central1 (Iowa)
  - **Zone:** us-central1-a
- **Disk source type:** Blank disk
- **Disk settings:** 
  - **Disk type:** Hyperdisk Balanced
  - **Size:** 100
  - **Provisioned IOPS:** 3600 (LEAVE TO DEFAULTS)
  - **Provisioned throughput:** 290 (LEAVE TO DEFAULTS)
- **Storage pool:** leave UNCHECKED  
- **Snapshot schedule (Recommended):**  leave empty 
- **Encryption:** Google managed encryption 
- Click on **CREATE**
```t
# Create Hyperdisk Balanced
gcloud compute disks create hyperdisk2 \
    --project=gcplearn9 \
    --type=hyperdisk-balanced \
    --description=hyperdisk1 \
    --size=100GB \
    --zone=us-central1-a \
    --provisioned-iops=3600 \
    --provisioned-throughput=290
```

## Step-03: Review the Hyperdisk
- Go to Compute Engine -> Storage -> Disks -> hyperdisk1

## Step-04: Clean-Up
```t

# Delete Hyperdisk
gcloud compute disks delete hyperdisk1 --zone us-central1-a
```
## Additional Reference
- https://cloud.google.com/compute/docs/disks/hyperdisks

---

Here is a well-structured `README.md` capturing the full context of your **Hyperdisk** demo in Google Cloud:


#  Google Cloud Hyperdisk - Hands-On Demo

This guide explores **Hyperdisks** in Google Compute Engine, how they differ from Persistent Disks, and the use cases and constraints for each type of Hyperdisk.

---

## ✅ Step-by-Step Summary

1. Understand what **Hyperdisks** are and how they differ from traditional Persistent Disks.
2. Explore **disk provisioning options**: dedicated IOPS and throughput.
3. Review available **Hyperdisk types**: Balanced, Extreme, and Throughput.
4. Identify **machine type support** and **limitations** for each.
5. Prepare to create a Hyperdisk using GCP Console or CLI.

---

## 💡 Real-World Use (with Your Project Experience)

In your cloud architecture simulations, Hyperdisks help replicate **high-throughput, database-heavy, or scale-out analytics** workloads that require **fine-tuned performance configuration**—especially in performance benchmarking or cost-performance evaluations.

---

## 📦 Analogy to Remember

🧩 **Persistent Disk = Automatic Mode**  
Performance scales based on what size and CPU you pick.

🎮 **Hyperdisk = Manual Control Mode**  
You get to set the **exact IOPS and throughput**, just like tuning a racing car.

---

## 📊 Key Differences: Persistent Disk vs Hyperdisk

| Feature                        | Persistent Disk                    | Hyperdisk                              |
|-------------------------------|------------------------------------|----------------------------------------|
| Performance Scaling           | Auto (based on size/CPU)           | Manual (IOPS & throughput configurable)|
| Boot Disk Support             | Yes                                | ❌ Not for Extreme/Throughput types    |
| Regional Disk Support         | Yes                                | ❌ Only zonal                          |
| Image Creation Support        | Yes                                | ❌ Not supported for Extreme/Throughput|
| Cloning Support               | Yes                                | ❌ Not supported                       |
| Multi-Writer / Read-Only VMs  | Yes (in some types)                | ❌ Not supported                       |

---

## 🔧 CLI/Console Summary

### Create Hyperdisk via Console

1. Go to **Compute Engine → Disks → Create Disk**.
2. Set **Disk Type** to:
   - `Hyperdisk Balanced`
   - `Hyperdisk Extreme`
   - `Hyperdisk Throughput`
3. Enter:
   - **Size**
   - **Provisioned IOPS** (for Balanced & Extreme)
   - **Provisioned Throughput** (for Throughput & Balanced)
4. Select zone and create.

> NOTE: Supported only for specific machine types and high vCPU configurations.

---

## 📌 Hyperdisk Types & Use Cases

### 1. Hyperdisk Balanced
- **Use case**: Web apps, medium-tier DBs
- **Provision**: Size, IOPS, and throughput

### 2. Hyperdisk Extreme
- **Use case**: High-performance DBs, low latency
- **Provision**: Size, high IOPS & throughput
- **Limitation**: Only on C3, N2, and similar with ≥88 vCPUs

### 3. Hyperdisk Throughput
- **Use case**: Scale-out workloads (Kafka, Hadoop)
- **Provision**: Size and throughput (no IOPS tuning)

---

## 🎯 Interview Questions + Sample Answers

**Q1: When would you prefer a Hyperdisk over a Persistent Disk?**  
**A:** When you need precise control over IOPS and throughput (e.g., performance-critical DBs, batch analytics workloads), or when Persistent Disk auto-scaling doesn't meet performance requirements.

---

**Q2: Can Hyperdisks be used as boot disks?**  
**A:** Only **Hyperdisk Balanced** may support it under certain conditions. **Extreme and Throughput** Hyperdisks **cannot be used as boot disks**.

---

**Q3: Can Hyperdisks be cloned or used across multiple VMs?**  
**A:** No, cloning is not supported and they do not support multi-VM read or multi-writer modes.

---

## 📚 Additional References

- [GCP Hyperdisks Overview](https://cloud.google.com/compute/docs/disks/hyperdisk-overview)
- [Persistent Disk vs Hyperdisk](https://cloud.google.com/compute/docs/disks/performance)
- [Hyperdisk Machine Type Support](https://cloud.google.com/compute/docs/disks/hyperdisk-machine-type-support)




