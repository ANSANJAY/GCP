---
title: Google Cloud - Attach GPU to VM Instance
description: Learn to Attach GPU to VM Instance on Google Cloud Platform GCP
---
## Step-01: Introduction
- Create a VM with GPU attached to it

## Step-02: Request for GPU Quota
- Go to IAM & Admin -> Quotas & System Limits
- Go to Compute Engine API -> Search for **GPUS_ALL_REGIONS** 
- Select **GPU (all regions) -> EDIT QUOTA
- **New Value:** 1
- Click on **DONE**
- Click on **SUBMIT REQUEST**
- Verify email about approval request

## Step-03: Create a VM Instance with attached GPUs
- Go to Compute Engine -> VM Instances -> CREATE INSTANCE
- **Name:** gpu-vm1
- **Region:** us-central1
- **Zone:** us-central1-b
- **Machine Configuration:** GPU
- **GPU Type:** NVIDIA T4
- **Number of GPUs:** 1
- **BOOT DISK:** SWITCH IMAGE
- **Note:** The selected image requires you to install an NVIDIA CUDA stack manually. To skip manual setup, click "Switch Image" below to use a GPU-optimized Debian OS image with CUDA support at no additional cost.
- REST ALL LEAVE TO DEFAULTS
- click on **CREATE**

## Step-04: Review VM Details
- Go to Compute Engine -> VM Instances -> gpu-vm1 -> DETAILS TAB
- Review **Machine configuration** for GPU
- Review Boot disk, we will find deeplearning Image

## Step-05: Clean-Up
- Go to Compute Engine -> VM Instances -> gpu-vm1 -> DELETE
---

---
title: Google Cloud Machine Families & Machine Types
description: Understand Compute Engine machine families, predefined types, and custom configurations for optimized workload deployment.
---

## ✅ Step-by-Step Summary

1. GCP offers two major **Compute Engine categories**:
   - **General Purpose**: For flexible, cost-effective, and scalable workloads.
   - **Workload Optimized**: For compute-intensive, memory-heavy, or GPU-based workloads.

2. **General Purpose Machine Families**:
   - **E2** (Cost-optimized): Web servers, dev/test, low-traffic workloads.
   - **N2 / N2D** (Balanced): Apps, databases, and general enterprise use.
   - **T2D** (Scale-out): High-throughput web servers, containerized microservices.

3. **Workload Optimized Families**:
   - **C2 / C2D / C3**: Compute-optimized for HPC, modeling, gaming.
   - **M1 / M2 / M3**: Memory-optimized for SAP HANA, analytics, caching.
   - **A2 / A3**: Accelerator-optimized (GPU) for ML and HPC.

4. Each machine family contains **predefined machine types**, such as:
   - `e2-micro`, `e2-standard-2`, `n2-standard-4`, etc.
   - These types vary in vCPU count, memory, max disk capacity, and egress bandwidth.

5. If no predefined type suits your workload, you can choose a **Custom Machine Type**:
   - Define your own vCPU and memory combination.
   - Supported only in select families like `E2`, `N2`, `N2D`.

6. In the **VM creation console**, machine family selection influences:
   - CPU/memory options
   - GPU compatibility
   - Pricing model
   - Availability of custom configuration

7. GCP charges based on **vCPU count and memory provisioned**, whether it's a predefined or custom machine.

---

## 🛠 Real-World Use (with Your Project Experience)

> While building CI/CD runners at, we used `e2-standard-2` for general-purpose pipelines and later migrated to `n2-highmem` VMs for a database caching layer to handle increased volume.
>
> - For GPU ML workloads, we evaluated `a2-highgpu-1g` during performance tuning experiments.
> - Where predefined configs didn’t meet our compute-to-memory ratio needs, we used **Custom Machine Types** with 6 vCPUs and 48GB memory to optimize performance and cost.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What are machine families in GCP Compute Engine?**  
**A:** Machine families group VM types based on workload characteristics. GCP provides general purpose families (like `E2`, `N2`) and workload-optimized ones (like `C2` for compute, `M2` for memory, `A2` for GPUs).

---

**Q2: When would you use a custom machine type?**  
**A:** When the predefined configurations don’t match your application's CPU-to-memory ratio. For example, if you need 6 vCPUs and 48GB RAM, and no preset offers that, a custom machine type is ideal.

---

**Q3: What is the difference between `E2`, `N2`, and `T2D`?**  
**A:**
- `E2`: Budget-friendly, for general workloads like dev/test.
- `N2`: Balanced performance and price.
- `T2D`: Optimized for scale-out workloads with high throughput needs.

---

**Q4: Can all machine families support custom configurations?**  
**A:** No. Only select families like `E2`, `N2`, and `N2D` support custom machine types. Families like `C2` or `M1/M2` are only available in predefined configurations.

---

## 🧠 Analogy to Remember

🛒 **Shoe Store Analogy**:  
Predefined machine types are like standard shoe sizes (S, M, L). They work for most people. But if you need a perfect fit (say, size 10.5 wide), you go custom. GCP offers both off-the-shelf VMs and custom-made ones depending on your workload's “foot size.”

---

## 🔧 CLI Command Summary

```bash
# List all available machine types in a zone
gcloud compute machine-types list --filter="zone:us-central1-a"

# Create a predefined E2 standard VM
gcloud compute instances create demo-e2-vm \
  --zone=us-central1-a \
  --machine-type=e2-standard-2

# Create a custom VM with 6 vCPUs and 48 GB RAM
gcloud compute instances create custom-vm \
  --zone=us-central1-a \
  --custom-cpu=6 \
  --custom-memory=48GB \
  --machine-type=e2-custom-6-49152
```
---

---
title: Google Cloud Compute Engine – GPU Concepts
Description: Learn how GPUs are used in Compute Engine for ML, HPC, and 3D workloads, and understand supported machine families, OS images, and host maintenance behavior.
---

## ✅ Step-by-Step Summary

1. **GPU (Graphics Processing Unit)** is used in GCP Compute Engine for:
   - Machine Learning (ML)
   - Scientific computing (math-heavy workloads)
   - 3D visualization and graphics rendering

2. To use a GPU with a VM:
   - Select a **GPU-compatible machine family**
   - Choose a **supported GPU type** (e.g., NVIDIA T4, A100)
   - Configure the **number of GPUs**
   - Use a **supported boot disk image** such as **Deep Learning VM Image**, not standard Debian/Ubuntu

3. **Supported Machine Families**:
   - `n1` (General Purpose): GPUs can be **attached during or after** VM creation
   - `a2`, `a3`, `g2` (Accelerator-Optimized): GPUs are **pre-attached only during creation**, cannot be added later

4. **GPU Support Across VM Models**:
   - Supported in **Standard**, **Spot**, and **Preemptible** VMs
   - **Billing** starts per-minute once GPU is attached

5. **Host Maintenance Behavior**:
   - **Live migration is not supported** for GPU VMs
   - GPU-attached VMs are **stopped** during host maintenance by default

6. **GPU and Block Storage**:
   - You can attach **local SSDs** to GPU-enabled VMs
   - But **not all GPU types support local SSDs**, check compatibility first

---

## 🛠 Real-World Use (with Your Project Experience)

> During a GPU benchmarking project at  for an ML pipeline:
>
> - We created `a2-highgpu-1g` instances with the **Deep Learning VM image** preloaded with CUDA and TensorFlow.
> - We used **spot VMs** with GPUs to reduce training costs.
> - When testing general-purpose `n1-standard-8` VMs, we manually attached a T4 GPU after creation.
> - A cronjob backed up training checkpoints to GCS because GPU VMs could not live migrate during maintenance.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What kind of workloads require GPU support in Compute Engine?**  
**A:** Machine learning, scientific computing, and 3D rendering tasks benefit from GPUs due to their parallel processing capabilities.

---

**Q2: Can you add a GPU to an existing VM?**  
**A:** Yes, but only if it's a VM from the `n1` family. For `a2`, `a3`, and `g2`, GPUs must be selected during initial VM creation.

---

**Q3: Can GPU-enabled VMs live migrate during host maintenance?**  
**A:** No. GPU VMs cannot live migrate. They are stopped when their host undergoes maintenance.

---

**Q4: Are GPUs supported in Spot and Preemptible VMs?**  
**A:** Yes, GPUs can be attached to both Spot and Preemptible instances, making GPU workloads more affordable for short-term or fault-tolerant tasks.

---

**Q5: Can I use local SSDs with GPU-enabled VMs?**  
**A:** Yes, but only certain GPU types support local SSDs. You must check compatibility in the documentation.

---

## 🧠 Analogy to Remember

🎮 **Supercharged Workstation Analogy**:
Think of a GPU VM like upgrading your laptop with a high-end graphics card. It lets you render video faster or train an ML model more efficiently — but when maintenance hits, you have to shut it down, save your work, and come back later.

---

## 🔧 CLI Command Summary

```bash
# List available GPU types
gcloud compute accelerator-types list --filter="zone:us-central1-a"

# Create a GPU VM (N1 family with manual GPU attachment)
gcloud compute instances create n1-gpu-vm \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --maintenance-policy=TERMINATE \
  --image-project=deeplearning-platform-release \
  --image-family=common-cu113 \
  --boot-disk-size=100GB

# Optional: Add a local SSD (check compatibility)
gcloud compute instances create gpu-vm-with-ssd \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --local-ssd interface=NVME \
  --maintenance-policy=TERMINATE \
  --image-project=deeplearning-platform-release \
  --image-family=common-cu113 \
  --boot-disk-size=100GB
```
---

   
