---
title: Google Cloud Create VM instance with Machine Images
description: Learn to Create VM instance with Machine Images on Google Cloud Platform GCP
---

## Step-01: Introduction
- A machine image contains a VM’s properties, metadata, permissions, and data from all its attached disks. 
- You can use a machine image to create, backup, or restore a VM.
### Usecase
1. Create a VM Instance
2. We will create a `Machine Image` from a `VM Instance` using Google Cloud Console
3. We will create a `VM Instance` from newly created `Machine Image` and verify
4. We will create a `Machine Image` using `gcloud`
5. Delete Machine Images and VM Instances

## Step-02: Create a VM Instance
```t
# Set Project
gcloud config set project PROJECT_ID
gcloud config set project gcplearn9

# Create VM Instance
gcloud compute instances create demo5-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --network-interface=subnet=default \
  --tags=http-server 

# Connect to VM Instance using SSH browser
demo5-vm -> SSH -> Open in new browser window

# Upload webserver-install.sh
Upload file webserver-install.sh

# Execute webserver-install.sh
chmod 755 webserver-install.sh
./webserver-install.sh
curl localhost
sudo echo "Machine Images Demo 101" | sudo tee /var/www/html/midemo.html
curl localhost/midemo.html

# Access Application 
http://<external-ip-of-vm>
http://<external-ip-of-vm>/midemo.html

# Make a note of VM Instance Hostname and IP Address
Hostname: demo5-vm
IP Address: 10.128.0.16
```


## Step-03: Create Machine Image
- Go to Compute Engine -> VM Instances -> demo5-vm -> Create new machine Image
- **Name:** demo5-vm-machine-image
- **Description:** demo5-vm-machine-image
- **Source VM Instance:** demo5-vm  (Auto-selected)
- **Location:** Multi-regional
- **Select Location:** us (United States)
- **Encryption:** Google Managed Encryption Key (leave to default)
- **Review Advanced Information** 
  - Machine type
  - Architecture
  - Network tags
  - Custom metadata
  - Service accounts
  - Deletion protection
  - Boot disk and local disks
  - Additional disks
  - Network interfaces
  - Available policies
    - Preemptibility
    - Automatic restart
    - On host maintenance
- Click on **Create**
- **Observation:** It will take 5 to 10 minutes to create the Machine Image. 

## Step-04: Verify the Properties after creation of Machine Image
- Go to Compute Engine -> Machine Images -> demo5-vm-machine-image
- Review **Properties**

## Step-05: Create VM Instance from Machine Image
- **Option-1:** Go to Compute Engine -> VM Instances -> Create Instance -> New VM instance from Machine Image
- **Option-2:** Go to Compute Engine -> Machine Images -> demo5-vm-machine-image -> Actions -> Create Instance
- **Name:** demo5-vm-from-machine-image
- Rest all verify if all settings loaded from Machine Image
- Click on **Create**


## Step-06: Verify by accessing webserver pages for demo5-vm-from-machine-image
- If required, you can do SSH connect and verify
```t
# Access Webserver Pages
http://<external-ip-of-vm>
http://<external-ip-of-vm>/midemo.html

Observation:
1. index.html content should be same as demo5-vm
2. midemo.html content should be same as demo5-vm
3. Using machine images we have installed once and reusing the same image for creating identical vms
```  
- **Key Observation:** VM Hostname and VM Private IP in static pages of a webserver will be from `demo5-vm` because it is almost a VM Instance clone using Machine Image concept. 

## Step-07: Create Machine Image using gcloud cli
```t
# Create Machine Image
gcloud compute machine-images create demo5-vm-machine-image-gcloud \
  --source-instance=demo5-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us  

# List Machine Images
gcloud compute machine-images list  

# Delete Machine Images
gcloud compute machine-images delete demo5-vm-machine-image
gcloud compute machine-images delete demo5-vm-machine-image-gcloud
```

## Step-08: Stop VM to avoid charges
- Go to VM Instances -> demo5-vm-from-machine-image  -> Delete
- Go to VM Instances -> demo5-vm -> Delete
```t
# Delete VM instances
gcloud compute instances delete demo5-vm --zone us-central1-a
gcloud compute instances delete demo5-vm-from-machine-image --zone us-central1-a
```

## Additional References
- [Machine Images](https://cloud.google.com/compute/docs/machine-images)
- [Creating Machine Images](https://cloud.google.com/compute/docs/machine-images/create-machine-images)

---
title: Google Cloud Create VM instance with Machine Images
description: Learn to Create VM instance with Machine Images on Google Cloud Platform GCP
---

## Step-01: Introduction
- A machine image contains a VM’s properties, metadata, permissions, and data from all its attached disks. 
- You can use a machine image to create, backup, or restore a VM.

### Usecase
1. Create a VM Instance
2. We will create a `Machine Image` from a `VM Instance` using Google Cloud Console
3. We will create a `VM Instance` from newly created `Machine Image` and verify
4. We will create a `Machine Image` using `gcloud`
5. Delete Machine Images and VM Instances

---

## ✅ Step-by-Step Summary

1. Provisioned a VM named `demo5-vm` using `e2-micro` machine type
2. Connected via SSH and manually installed a web server with a custom HTML page
3. Captured a **machine image** of the VM using the Console
4. Verified machine image metadata and configuration settings
5. Created a new VM (`demo5-vm-from-machine-image`) from the machine image
6. Confirmed that both VMs served the same static content
7. Repeated the process using the `gcloud` CLI
8. Deleted unused resources to prevent cost

---

## 🛠 Real-World Use (with Your Project Experience)

At American Express, I implemented an automation pipeline that involved scanning code quality using GitHub Actions and SonarQube. To scale this efficiently, I:

- Created a base VM with GitHub CLI, SonarQube scanners, and required security tools installed.
- Captured a **machine image** from this base VM.
- Used this image to rapidly spin up ephemeral VMs for different teams' GitHub PRs, reducing provisioning time from 3–5 minutes to under 30 seconds.
- Used **multi-regional storage** for cross-region accessibility and compliance.

This ensured uniform configuration across environments and simplified recovery and scaling.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What’s the benefit of using a machine image over a startup script?**  
A: A machine image contains both the configuration and data of a VM, allowing for instant provisioning. Startup scripts require repeated setup on boot, increasing VM launch time.

**Q2: What does a machine image include and exclude?**  
A: It includes VM metadata, disk data, and network config. It excludes in-memory data, local SSD data, and original IP or VM name.

**Q3: How would you use machine images in a CI/CD pipeline?**  
A: Pre-bake scanners and dependencies into a base VM, create a machine image, then spin up VMs from that image for every CI/CD run. This reduces time, ensures consistency, and avoids config drift.

**Q4: Are machine images editable?**  
A: No. Machine images are immutable. You can launch a VM from it, make changes, and save a new image.

---

## 🧠 Analogy to Remember

📸 **Photocopy Analogy**:  
Imagine customizing a document by hand (installing software on a VM). Once perfected, you photocopy it (machine image). Every new copy is instant and exactly the same, no need to rewrite or reformat.

---

## 🔧 CLI Command Summary

```bash
# Set project
gcloud config set project gcplearn9

# Create VM Instance
gcloud compute instances create demo5-vm \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --network-interface=subnet=default \
  --tags=http-server 

# Create Machine Image via CLI
gcloud compute machine-images create demo5-vm-machine-image-gcloud \
  --source-instance=demo5-vm \
  --source-instance-zone=us-central1-a \
  --storage-location=us  

# List Machine Images
gcloud compute machine-images list  

# Delete Machine Images
gcloud compute machine-images delete demo5-vm-machine-image
gcloud compute machine-images delete demo5-vm-machine-image-gcloud

# Delete VM Instances
gcloud compute instances delete demo5-vm --zone us-central1-a
gcloud compute instances delete demo5-vm-from-machine-image --zone us-central1-a

