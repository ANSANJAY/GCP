---
title: Google Cloud Compute Engine Hyperdisks with Storage Pools
description: Learn to create Compute Engine Hyperdisks storage pools
---

## Step-01: Introduction
- Understand Hyperdisk Storage pools
- Review the options to create Storage pool
- **Important Note:** We are not going to create storage pool, it is going to charge us heavily, so we will only explore the options to create it

## Step-02: Create Storage Pool
- Go to Compute Engine -> Storage -> Storage Pool
- **Name:** storage-pool-1
- **Location:**
  - **Region:** us-central1
  - **Zone:** us-central1-a
- **Pool type:** Hyperdisk Balanced
- **Capacity Type:** Advanced capacity
- **Storage pool capacity:** 10GB
- **Provisioned IOPs:** 10000
- **Provisioned Throughput:** 1024
- Click on **SUBMIT**
```t
# Create Compute Engine Storage Pool
gcloud compute storage-pools create storage-pool-1 \
  --project=gcplearn9 \
  --provisioned-capacity=10240 \
  --storage-pool-type=hyperdisk-balanced \
  --zone=projects/gcplearn9/zones/us-central1-a \
  --provisioned-iops=10000 \
  --provisioned-throughput=1024
```
 
## Additional Reference
- https://cloud.google.com/compute/docs/disks/storage-pools


# 📦 Google Cloud Hyperdisk Storage Pools - Hands-On Guide


---

## ✅ Step-by-Step Summary

1. Learn what **Hyperdisk Storage Pools** are and when to use them.
2. Understand **thin provisioning**, **auto-grow**, and **data reduction**.
3. Review **use cases**, **limitations**, and **pool types**.
4. Walk through how to create a storage pool and attach a Hyperdisk to it.

---

## 💡 Real-World Use (with Project Application)

In scenarios involving **massive VM deployments** (e.g., analytics, backups, or on-prem SAN migrations), you can:
- **Pre-purchase IOPS + throughput + capacity**
- Share this pool across multiple disks
- Reduce cost via **thin provisioning** and **auto-grow**

This model helps minimize unused storage across thousands of block devices.

---

## 🎮 Analogy to Remember

🛒 **Storage Pool = Warehouse Bulk Order**  
Instead of buying one product at a time, you buy in bulk and distribute as needed.

🧱 **Hyperdisks = Custom Boxes**  
You allocate exactly what each "box" (disk) needs from the pool: space, throughput, and IOPS.

---

## ⚙️ What Is a Storage Pool?

A **Storage Pool** is a zonal resource that pre-purchases:
- **Capacity** (in TB)
- **Provisioned IOPS**
- **Provisioned Throughput (MB/s)**

You can then create **Hyperdisks** that consume resources from this pool.

---

## 🧠 Key Features

| Feature                  | Description                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| Thin Provisioning        | Disks only consume capacity they actually use, not what's allocated         |
| Data Reduction           | Compression reduces space usage                                            |
| Auto-Grow                | Automatically expands capacity when pool usage exceeds 80%                 |
| VM Lifecycle Independence| Disks persist even if VM is deleted                                       |

---

## 🧪 Pool Types

### 1. **Hyperdisk Balanced Pool**
- Define: **Capacity + IOPS + Throughput**
- Use for: Web apps, medium DBs, custom I/O tuning

### 2. **Hyperdisk Throughput Pool**
- Define: **Capacity + Throughput**
- Use for: Scale-out analytics (e.g., Hadoop, Kafka)

---

## 📊 CLI/Console: Create a Storage Pool

### Via Console

1. Go to **Compute Engine → Storage → Storage Pools → Create**.
2. Enter:
   - Pool name
   - Zone
   - Pool type (Balanced / Throughput)
   - Capacity (e.g., 10 TB)
   - Provisioned IOPS and/or Throughput
3. Submit *(⚠️ Costly! Avoid in test environments)*

### Via GCloud (Not Run Here)
```bash
gcloud compute storage-pools create my-pool \
  --zone=us-central1-a \
  --pool-type=hyperdisk-balanced \
  --provisioned-capacity=10TB \
  --provisioned-iops=40000 \
  --provisioned-throughput=1000
````

---

## 📦 Creating a Hyperdisk from Storage Pool

1. Go to **Compute Engine → Disks → Create Disk**.
2. Choose:

   * **Disk Type:** Hyperdisk (Balanced / Throughput)
   * **Associate to Storage Pool**
3. Set:

   * Disk size
   * Provisioned IOPS & Throughput
4. Create a disk – it draws resources from your storage pool.

---

## 🚨 Limitations

| Constraint                  | Description                                               |
| --------------------------- | --------------------------------------------------------- |
| Zonal Only                  | Regional pools not supported                              |
| Compute Engine Only         | Cloud SQL and others not supported                        |
| Max 1 PiB Capacity per Pool | Per storage pool                                          |
| Max 1000 Disks per Pool     | Scalability limit                                         |
| No Clone/Image Support      | You cannot clone disks or use them in snapshots/templates |
| No Multi-VM Attachments     | No read-only or multi-writer across VMs                   |

📎 [GCP Docs - Hyperdisk Storage Pools Limitations](https://cloud.google.com/compute/docs/disks/hyperdisk-storage-pools#limitations)

---

## 🧭 When to Use Storage Pools

1. **On-Prem SAN Migrations**: Moving large storage from SAN to GCP.
2. **Underutilized Disks**: When provisioned storage exceeds real usage.
3. **Large-scale Deployments**: Managing thousands of disks becomes easier.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is the benefit of using a Storage Pool over traditional provisioning?**
**A:** It allows thin provisioning, where unused disk space is not wasted. It also supports auto-grow and centralized performance tuning for multiple Hyperdisks.

---

**Q2: Can you use Storage Pools for regional disks or Cloud SQL?**
**A:** No. Storage Pools are zonal and can only be used with Compute Engine.

---

**Q3: How does thin provisioning help?**
**A:** It allows a disk to reserve 100 GB but only consume 5 GB from the pool if that’s all it’s actually using, preventing over-provisioning waste.

---

## 📚 Additional Reference

* [GCP Docs - Hyperdisk Storage Pools](https://cloud.google.com/compute/docs/disks/hyperdisk-storage-pools)
* [GCP Hyperdisks Overview](https://cloud.google.com/compute/docs/disks/hyperdisk-overview)

---

## 🧼 Cleanup Strategy

* Delete all Hyperdisks first
* Then delete the Storage Pool to prevent incurring ongoing costs

---


