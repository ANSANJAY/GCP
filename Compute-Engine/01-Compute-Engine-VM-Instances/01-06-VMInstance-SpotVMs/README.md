---
title: Google Cloud Create VM instance with Spot VMs
description: Learn to Create VM instance with Spot VM Option
---

## Step-01: Introduction
- **Google Cloud Console:** Create VM instance using VM Provisioning Model: SPOT 
- **gcloud CLI:** Create VM instance using VM Provisioning Model: SPOT 

## Step-02: Create a VM Instance
- Go to Virtual Machines -> VM Instances -> Create Instance -> New VM Instance -> Provide required details
- **Name:** demo6-vm-spot
- **Labels:** environment: dev
- **Region:** us-central1
- **Zone:** us-central1-a 
- **Machine Configuration:** 
  - **Series:** E2
  - **Machine Type:** e2-micro
- **Availability Policies:**
  - **VM Provisioning Model:** SPOT  
  - **On VM termination:** STOP or DELETE
- **Display Device:** checked 
- **Confidential VM Service:** unchecked (leave to default)
- **Container:** unchecked (leave to default)
- **Boot Disk:** Leave to defaults
- **Identity and API Access:**
  - **Service Account:** Compute Engine default Service Account
  - **Access Scopes:** Allow default Access
- **Firewall**
  - Allow HTTP Traffic
- **Advanced Options**
  - **Management:**
    - **Description:** demo2-vm-startupscript
    - **Deletion Protection:** Enable it
    - **Reservations:** Automatically use created reservation (leave to default)
    - **Automation:**
      - **Startup Script:** Copy paste content from `webserver-install.sh` file
```t
#!/bin/bash
sudo apt install -y telnet
sudo apt install -y nginx
sudo systemctl enable nginx
sudo chmod -R 755 /var/www/html
HOSTNAME=$(hostname)
sudo echo "<!DOCTYPE html> <html> <body style='background-color:rgb(250, 210, 210);'> <h1>Welcome to StackSimplify - WebVM App1 </h1> <p><strong>VM Hostname:</strong> $HOSTNAME</p> <p><strong>VM IP Address:</strong> $(hostname -I)</p> <p><strong>Application Version:</strong> V1</p> <p>Google Cloud Platform - Demos</p> </body></html>" | sudo tee /var/www/html/index.html
```
  - **Metadata:** empty (leave to default)
  - **Data Encryption:** Leave to defaults
- Click on **Create**

## Step-03: Verify the VM
- Go to VM Instances -> demo6-vm-spot -> Details Tab
- Review the warning `Spot VMs may be terminated at any time`
- Go to **Availability policies** and Verify `VM provisioning model`

## Step-04: Access Application
```t
# Access Application
http://<EXTERNAL-IP-OF-VM>
```

## Step-05: Delete VM
```t
# Delete VM Instance
gcloud compute instances delete demo6-vm-spot --zone us-central1-a
```

## Step-06: Create Spot VM Instance using gcloud
```t
# Set Project
gcloud config set project PROJECT_ID
gcloud config set project gcplearn9

# Create VM Instance
# --instance-termination-action: Options  STOP OR DELETE
gcloud compute instances create demo6-vm-spot-gcloud \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --network-interface=subnet=default \
  --tags=http-server \
  --metadata-from-file=startup-script=webserver-install.sh     

# Access Application 
http://<external-ip-of-vm>

# Delete VM Instance
gcloud compute instances delete demo6-vm-spot-gcloud --zone us-central1-a
```

## Additional References
- [Spot VMs](https://cloud.google.com/compute/docs/instances/spot)


---
title: Google Cloud VM Live Migration and Availability Policies
description: Understand Live Migration, On-Host Maintenance Options, and Automatic Restart Policies in GCP Compute Engine
---

## ✅ Step-by-Step Summary

1. **VMs run on physical host machines** managed by Google Cloud.
2. GCP performs **infrastructure maintenance** (hardware/software/network upgrades) on those hosts.
3. You can configure how VMs behave during maintenance using **Availability Policies**.
4. These settings apply only to **Standard VMs**, not to **Spot or Preemptible VMs**.
5. Two key options in Availability Policy:
   - **On-Host Maintenance**
     - `Migrate VM instance` (Recommended)
     - `Terminate VM instance`
   - **Automatic Restart**
     - If enabled, the VM restarts automatically after unexpected stops (e.g., host failure).
6. Live migration allows VMs to move **without downtime** across hosts within the **same zone**.
7. Even VMs with **local SSDs** are supported (except for those using GPUs).
8. These settings are configured when you create or update a VM instance under **Advanced > Availability Policy**.

---

## 🛠 Real-World Use (with Your Project Experience)

> While managing CI/CD runners for SonarQube integration at AmEx, we used **Standard VMs** with **Live Migration** enabled. This ensured that GCP host-level maintenance didn't interrupt active PR scan jobs.
>
> - We configured **Automatic Restart = ON** to recover from underlying failures.
> - Our critical agents were set to `Migrate VM Instance`, allowing seamless transitions with no job failures.
> - This helped maintain system reliability and reduced operational overhead during weekly GCP maintenance windows.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is live migration in GCP?**  
**A:** Live migration allows Google Compute Engine to move running VM instances between host machines during maintenance without shutting them down or restarting. It ensures zero downtime.

---

**Q2: Which VM types support live migration?**  
**A:** Live migration is supported for **Standard VMs** and **VMs with local SSDs**, but not for **Spot**, **Preemptible**, or **GPU-attached** instances.

---

**Q3: What are the options under On-Host Maintenance policy?**  
**A:** You can choose to either:
- `Migrate VM instance` – live migration to another host (recommended)
- `Terminate VM instance` – stops the VM during host maintenance

---

**Q4: What is the purpose of the Automatic Restart option?**  
**A:** It defines whether a VM should automatically restart if it is terminated due to non-user initiated reasons (like host failure). Enabling this ensures high availability.

---

## 🧠 Analogy to Remember

🧳 **Hotel Guest Analogy**:
- You’re staying in a hotel (VM on a host).  
- If the hotel needs maintenance (GCP host maintenance), they **quietly move you to another room** (live migration) **without waking you up**.
- If your room is unavailable after the move (host failure), the system will **book you into another room automatically** (automatic restart).

---

## 🔧 CLI Command Summary

```bash
# Create a standard VM with live migration and auto restart
gcloud compute instances create live-mig-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --maintenance-policy=MIGRATE \
  --restart-on-failure \
  --provisioning-model=STANDARD

# View availability policy settings
gcloud compute instances describe live-mig-vm \
  --zone=us-central1-a \
  --format="get(scheduling.automaticRestart, scheduling.onHostMaintenance)"

```
---

---
title: Google Cloud Spot VMs & Preemptible VMs
description: Learn how to use Spot VMs and understand the differences from legacy Preemptible VMs in GCP. Optimize cost for fault-tolerant workloads.
---

## ✅ Step-by-Step Summary

1. **Spot VMs** are temporary virtual machines that use GCP's **excess Compute Engine capacity**.
2. They offer a **60–91% discount** compared to standard VMs.
3. Spot VMs are **subject to preemption**—GCP may stop or delete them **at any time** to reclaim resources.
4. **No automatic restart** or **live migration** support is available for Spot VMs.
5. You can configure **On VM Termination** action as:
   - `STOP` (default)
   - `DELETE`
6. You receive a **30-second warning** before a Spot VM is preempted.
7. **Shutdown scripts** can be added to gracefully handle preemption events.
8. **Use Spot VMs** for **fault-tolerant and cost-sensitive workloads**:
   - HPC
   - Big data analytics
   - CI/CD runners
   - Batch processing
   - GKE node pools
9. **Preemptible VMs** are an older version of Spot VMs:
   - Max runtime of **24 hours**
   - **No GPU or local SSD support**
   - Not recommended for new use cases

---

## 🛠 Real-World Use (with Your Project Experience)

> During batch-based artifact verification at AmEx, we used **Spot VMs** to reduce infrastructure costs by up to **75%**.
> 
> - Jobs were checkpointed using GCS and Cloud SQL.
> - Spot VMs ran as **ephemeral CI/CD runners**, where interruptions were tolerable.
> - Preemption-aware shutdown scripts archived logs and uploaded partial data before deletion.
> - For fault-tolerant GKE clusters, **Spot node pools** helped auto-scale across zones.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What are Spot VMs in GCP?**  
**A:** Spot VMs are temporary virtual machines that run on GCP's unused capacity and are available at a discounted price. They can be preempted anytime if GCP needs the resources.

---

**Q2: How do Spot VMs compare to Preemptible VMs?**  
**A:** Spot VMs are a modern replacement for Preemptible VMs. Unlike Preemptible VMs, Spot VMs:
- Have no 24-hour limit
- Support GPUs and local SSDs
- Offer configurable termination behavior (Stop/Delete)

---

**Q3: What is the termination behavior of Spot VMs?**  
**A:** You can choose to:
- `STOP` the VM (default), allowing reuse when capacity is available
- `DELETE` the VM permanently

Both options receive a 30s warning before termination.

---

**Q4: Can I use free tier credits with Spot VMs?**  
**A:** No. Spot VMs are **excluded** from GCP's free tier credits. A **billing-enabled project** is required.

---

**Q5: What kind of workloads are best for Spot VMs?**  
**A:** Fault-tolerant and parallel workloads such as:
- Data processing jobs
- CI/CD runners
- ML training (checkpointed)
- GKE node pools with auto-healing

---

## 🧠 Analogy to Remember

🎟️ **Last-Minute Movie Ticket Analogy**:  
Spot VMs are like cheap last-minute movie tickets for empty seats. You get a great deal, but you could be asked to leave if the theater needs the seat back. They're perfect if you're flexible and don’t mind interruptions.

---

## 🔧 CLI Command Summary

```bash
# Create Spot VM with STOP termination policy (default)
gcloud compute instances create spot-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --provisioning-model=SPOT \
  --instance-termination-action=STOP

# Create Spot VM with DELETE termination policy
gcloud compute instances create spot-vm-del \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --provisioning-model=SPOT \
  --instance-termination-action=DELETE

# Add shutdown script to Spot VM
gcloud compute instances add-metadata spot-vm \
  --metadata shutdown-script='#!/bin/bash
echo "VM preempted at $(date)" >> /var/log/preempt.log'

# Describe the instance's provisioning model and termination behavior
gcloud compute instances describe spot-vm \
  --format="get(scheduling.provisioningModel, scheduling.instanceTerminationAction)"
```
