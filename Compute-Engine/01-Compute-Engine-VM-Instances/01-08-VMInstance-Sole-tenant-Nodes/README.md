---
title: Google Cloud - Sole-tenant Nodes
Description: Learn to create GCP GCE sole-tenant Nodes
---
## Step-01: Introduction
- Explore creating sole-tenant nodes using Google Cloud Console

## Step-02: Create a Sole-tenant Node
- Go to Compute Engine -> Virtual machines -> Sole-tenant Nodes -> CREATE NODE GROUP
### Node group properties
- **Name:** node-group-1
- **Region:** us-central1
- **Zone:** us-central1-a
- Click on **CONTINUE**
### Node template properties
- Click on **CREATE NODE TEMPLATE**
- **Name:** node-template-1
- **Node type:** c2-node-60-240
- **Local SSD:** none
- **GPU accelerator:** none
- **Affinitly Lables**
  - **Key:** appname
  - **Value:** mybankingapps
- Click on **CREATE**
- Click on **CONTINUE**
### Configure autoscaling
- **Autoscaling mode:** on
- **Minimum Number of nodes:** 1
- **Maximum Number of nodes:** 3
- Click on **CONTINUE**
### Configure maintenance settings (optional)
- **Maintenance policy:** Default (RECOMMENDED)
- **Maintenance window:** 0:00 - 4:00
- Click on **CONTINUE**
### Configure share settings (optional)
- Select **Do not share this node group with other projects**
- Click on **CREATE** DONT CREATE IT -  WE ARE JUST EXPLORING

## Step-03: Delete the Node Template 
- Go to Compute Engine -> Virtual machines -> Sole-tenant Nodes -> Node Templates
- Delete **node-template-1**

## Step-04: gcloud: Sole-tenant Node Commands
```
# List Sole-tenant node types
gcloud compute sole-tenancy node-types list

# List Sole-tenant node templates
gcloud compute sole-tenancy node-templates list

# List Sole-tenant node groups
gcloud compute sole-tenancy node-groups list

# Create a sole-tenant node template
# https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms#creating-a-sole-tenant-node-template
gcloud compute sole-tenancy node-templates create TEMPLATE_NAME \
  --node-type=NODE_TYPE \
  [--region=REGION \]
  [--node-affinity-labels=AFFINITY_LABELS \]
  [--accelerator type=GPU_TYPE,count=GPU_COUNT \]
  [--disk type=local-ssd,count=DISK_COUNT,size=DISK_SIZE \]
  [--cpu-overcommit-type=CPU_OVERCOMMIT_TYPE]

# Create a sole-tenant node group
# https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms#create_a_sole-tenant_node_group
gcloud compute sole-tenancy node-groups create GROUP_NAME \
  --node-template=TEMPLATE_NAME \
  --target-size=TARGET_SIZE \
  [--zone=ZONE \]
  [--maintenance-policy=MAINTENANCE_POLICY \]
  [--maintenance-window-start-time=START_TIME \]
  [--autoscaler-mode=AUTOSCALER_MODE: \
  --min-nodes=MIN_NODES \
  --max-nodes=MAX_NODES]  

# Provision a sole-tenant VM
# https://cloud.google.com/compute/docs/nodes/provisioning-sole-tenant-vms#provision_a_sole-tenant_vm
gcloud compute instances create VM_NAME \
  [--zone=ZONE \]
  --image-family=IMAGE_FAMILY \
  --image-project=IMAGE_PROJECT \
  --node-group=GROUP_NAME \
  --machine-type=MACHINE_TYPE \
  [--maintenance-policy=MAINTENANCE_POLICY \]
  [--accelerator type=GPU_TYPE,count=GPU_COUNT \]
  [--local-ssd interface=SSD_INTERFACE \]
  [--restart-on-failure]  
```
---
title: Google Cloud Compute Engine – Sole-Tenant Nodes
Description: Learn how to configure and use Sole-Tenant Nodes in Compute Engine for physically isolated, high-compliance, and license-specific workloads.
---

## ✅ Step-by-Step Summary

1. **Sole-Tenant Nodes** provide a **dedicated physical server** (host) to run only your project's VM instances.
   - Unlike **multi-tenant nodes** (shared by multiple customers), sole-tenant nodes are **not shared** with any other GCP customers.

2. Use Cases:
   - Regulatory compliance (finance, healthcare)
   - Workloads with **licensing restrictions** (e.g., Windows BYOL)
   - **Performance isolation** for gaming, rendering, ML, or large-scale data processing

3. **All Compute Engine features are supported** on sole-tenant nodes (e.g., GPUs, SSDs, autoscaling).

4. **Components of Sole-Tenant Node Setup**:
   - **Node Template** (regional resource)
     - Defines node type (CPU/memory), GPU, SSD, affinity labels
     - Examples:
       - `m1-node-96-1433`: 96 vCPU, 1433 GB RAM
       - `c2-node-60-240`: 60 vCPU, 240 GB RAM
   - **Node Group**
     - A group of nodes within a single zone that act as a scheduling unit
     - Requires:
       - Node template reference
       - Region and zone
       - Autoscaling (min/max node count)
       - Maintenance settings
       - Sharing configuration (project-level)

5. **Affinity Labels**:
   - Used to **pin VMs to specific node groups** based on labels to ensure isolation.

6. **Maintenance Scheduling**:
   - Sole-tenant nodes allow **configurable maintenance windows**, unlike regular shared hosts.

7. **GPU & SSD Support**:
   - GPUs and local SSDs **can be added** to sole-tenant nodes via node templates.
   - Not all GPU types support local SSD — check documentation for compatibility.

8. **Sharing Settings**:
   - Control node group visibility across:
     - All projects in the organization
     - Only selected projects
     - Only current project (default)

---

## 🛠 Real-World Use (with Your Project Experience)

> In a regulatory reporting project at AmEx, we used **Sole-Tenant Nodes** to meet compliance requirements.
>
> - Workloads handling PII and regulatory-sensitive financial data were **isolated at the hardware level**.
> - We created node templates with `m1-node-96-1433` and reserved local SSDs for fast storage.
> - Node groups were autoscaled based on scheduled ETL load times.
> - A custom **maintenance window** ensured batch pipelines were unaffected by host updates.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is a sole-tenant node in GCP?**  
**A:** A sole-tenant node is a dedicated physical server used to host only your VMs, providing hardware-level isolation from other customers for compliance, licensing, and performance use cases.

---

**Q2: What are common use cases for sole-tenant nodes?**  
**A:** Healthcare or finance workloads requiring compliance, Windows licensing (BYOL), GPU-intensive machine learning jobs, and gaming workloads with predictable performance needs.

---

**Q3: How does a node template relate to a node group?**  
**A:** A node template defines the properties (CPU, memory, GPU, etc.) of a sole-tenant node. A node group uses a node template to create one or more sole-tenant nodes in a zone.

---

**Q4: Can you set a custom maintenance schedule for sole-tenant nodes?**  
**A:** Yes. Unlike regular Compute Engine VMs, sole-tenant nodes allow configurable maintenance windows to better align with operational downtime.

---

**Q5: Are local SSDs and GPUs supported in sole-tenant nodes?**  
**A:** Yes, but GPU type and local SSD compatibility must be verified. Some configurations may not support both together.

---

## 🧠 Analogy to Remember

🏢 **Office Floor Analogy**:
- A multi-tenant node is like renting one desk in a shared co-working space—cheap but shared with strangers.
- A sole-tenant node is like leasing an entire office floor—exclusive, isolated, and tailored to your needs (e.g., security, noise-free, compliance-ready).

---

## 🔧 CLI Command Summary

```bash
# Create a node template
gcloud compute sole-tenancy node-templates create m1-template \
  --node-type=m1-node-96-1433 \
  --region=us-central1 \
  --description="Node template with 96 vCPUs and 1433 GB RAM"

# Create a node group from the template
gcloud compute sole-tenancy node-groups create m1-node-group \
  --node-template=m1-template \
  --target-size=2 \
  --zone=us-central1-a

# Set affinity labels when creating VM (to pin it to node group)
gcloud compute instances create isolated-vm \
  --zone=us-central1-a \
  --machine-type=custom-4-16 \
  --node-group=m1-node-group \
  --node-affinity="key=env,operator=IN,values=prod"

# List available node types
gcloud compute sole-tenancy node-types list --region=us-central1
```

---
title: Google Cloud Compute Engine – Creating Sole-Tenant Nodes (Hands-On)
description: Learn how to create Sole-Tenant Node Templates and Node Groups using the Console and gcloud CLI. Understand configurations, policies, and best practices.
---

## ✅ Step-by-Step Summary

### Step 1: Navigate to Sole-Tenant Nodes
- Go to **VM Instances → Sole-Tenant Nodes**
- Click **Create Node Group**

### Step 2: Configure Node Group
- **Name**: `node-group-1`
- **Region**: `us-central1`
- **Zone**: `us-central1-a`

### Step 3: Create Node Template
- **Template Name**: `node-template-1`
- **Node Type**: `c2-node-60-240` (60 vCPUs, 240 GB RAM)
- **Local SSDs**: None
- **GPU**: None
- **Affinity Labels**:  
  - `env=banking`
- Click **Create**

### Step 4: Configure Autoscaling (Optional)
- **Autoscaling**: Enabled
- **Min Nodes**: 1
- **Max Nodes**: 3

### Step 5: Configure Maintenance Window
- **Policy**: `default`
- **Maintenance Time**: 2 AM – 4 AM

### Step 6: Configure Sharing (Optional)
- Choose whether to:
  - Share with no other projects (default)
  - Share with selected projects
  - Share with all projects in the org

> ❌ Note: Creation was **cancelled** in the demo to avoid high cost.

---

## 🛠 Real-World Use (with Your Project Experience)

> While provisioning sensitive financial apps at AmEx, we tested **sole-tenant node groups** for workloads requiring **hardware-level isolation**.
>
> - Used `c2-node-60-240` for better performance isolation.
> - Added affinity labels to ensure **compliance-pinned VMs** launched only on reserved hosts.
> - Auto-scaling controlled compute spend across peak/non-peak workloads.
> - Maintenance windows were scheduled post-midnight to minimize pipeline disruption.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is the difference between a node template and a node group?**  
**A:** A **node template** defines configuration like CPU, memory, GPU, and labels. A **node group** is a zonal group of nodes created using that template.

---

**Q2: Why use affinity labels in sole-tenant nodes?**  
**A:** To ensure certain VM workloads are deployed only on specific node groups, often for security, compliance, or licensing reasons.

---

**Q3: How does maintenance differ in sole-tenant nodes?**  
**A:** You can schedule **custom maintenance windows** for sole-tenant nodes, unlike multi-tenant hosts that follow Google's maintenance cycle.

---

**Q4: Can sole-tenant nodes be shared across projects?**  
**A:** Yes, you can configure share settings to expose the node group to all projects, or only selected ones in your organization.

---

## 🧠 Analogy to Remember

🏢 **Private Suite vs Hotel Room Analogy**:  
Using a multi-tenant host is like staying in a shared hotel. A sole-tenant node is like renting an entire private suite — complete control, no neighbors, and customized housekeeping (maintenance).

---

## 🔧 CLI Command Summary

### 🔹 List Node Types
```bash
gcloud compute sole-tenancy node-types list --region=us-central1
```

