---
title: Google Cloud -  Ops Agent
description: Learn to Install and Configure Ops Agent for Monitoring and Logging on VMs in Google Cloud Platform GCP
---

## Step-01: Introduction
- Install Ops Agent
- Verify Monitoring and Logging

## Step-02: Install Ops Agent 
### Step-02-01: Install Ops Agent using google cloud console
- Go to Compute Engine -> VM Instances -> CREATE INSTANCE
- **Name:** demo1-opsagent
- **Observability - Ops Agent:**
  - **Install Ops Agent for Monitoring and Logging:** CHECK IT
- Click on **CREATE**

### Step-02-02: Install Ops Agent manually
```t
# Creater VM Instance
gcloud compute instances create demo2-opsagent \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --network-interface=subnet=default \
  --tags=http-server \
  --metadata-from-file=startup-script=webserver-install.sh 

# Verify VM Metrics
1. Go to Compute Engine -> VM Instances -> demo2-opsagent -> OBSERVABILITY Tab
2. Click on CPU, PROCESS, MEMORY
3. It shows the message "Requires Ops Agent"

# Connect SSH to VM
gcloud compute ssh --zone "us-central1-a" "demo2-opsagent" --project "gcplearn9"

# Download Ops Agent
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh

# Install Ops Agent
sudo bash add-google-cloud-ops-agent-repo.sh --also-install
sudo apt list --installed | grep google-cloud-ops-agent

# Verify Ops Agent Status
sudo systemctl status google-cloud-ops-agent"*"

# Restarting Agent (Optional)
sudo service google-cloud-ops-agent restart

## Additional Optional Commands
# Upgrading Agent
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# Uninstalling the agent (For reference only)
sudo bash add-google-cloud-ops-agent-repo.sh --uninstall
```

## Step-03: Review current VM Monitoring Metrics in Observability Tab of VM
- Go to Compute Engine -> VM Instances -> demo2-opsagent -> Click on **Observability** tab
- Review Metrics in which many metrics says they need the Monitoring agent. 
  - CPU
  - Processes
  - Memory
  - Disk -> Capacity

## Step-05: Verify the Cloud Monitoring Tool
- Go to Monitoring -> Overview
- Click on **VIEW GCE DASHBOARD** 
- Click on **VMs Dashboard**
- Verify Agent Status 
  - Ops Agent 


## Step-06: Review the following 
- Go to Monitoring -> Overview
- Click on **VIEW GCE DASHBOARD** 
- Click on **VMs Dashboard**
- Review the following tabs
  - METRICS TAB
  - LOGS TAB

## Step-07: Verify Logs in Logs Explorer
- Go to Logging -> Logging
- **Resource Name:** demo2-opsagent
- Verify Logs

## Step-08: Cleanup - Delete VMs
- Delete VMs
  - demo1-opsagent
  - demo2-opsagent
 
---
## title: Google Cloud Compute Engine – Ops Agent Overview
## description: Understand what Ops Agent is, why it replaces legacy agents, and how it enhances observability on GCP VM instances for logging and monitoring.
---

## ✅ Step-by-Step Summary

1. **Ops Agent** is the recommended single agent for collecting **logs**, **metrics**, and **traces** from GCP VM instances.
   - Replaces:
     - Legacy Monitoring Agent
     - Legacy Logging Agent
   - New workloads should use **Ops Agent** exclusively.

2. **What Ops Agent Does**:
   - Collects logs → sends to **Cloud Logging**
   - Collects metrics/traces → sends to **Cloud Monitoring**
   - Uses:
     - `Fluent Bit` for logging
     - `OpenTelemetry Collector` for metrics/traces

3. **Key Features**:
   - YAML-based configuration for custom log/metrics collection
   - Supports both **Linux** and **Windows**
   - Installation options:
     - During VM creation (auto-install)
     - Manually via command line
     - Fleet install via **gcloud**, **Ansible**, **Puppet**, **Terraform**
     - From **Cloud Monitoring dashboard**

4. **Logging Features**:
   - Collects standard system logs
   - Collects **file-based logs** (custom paths)
   - Supports:
     - TCP/Forward protocols (Fluent Bit, Fluentd)
     - Parsing unstructured → structured logs (JSON, regex)
     - Filtering/exclusion rules via YAML
   - Supports third-party application logs (e.g., Kafka, nginx, MongoDB)

5. **Monitoring Features**:
   - Out-of-the-box collection for:
     - CPU, memory, disk, network, swap
     - Ops Agent health metrics
     - Windows-specific: IIS, MSSQL
     - Linux-only: GPU and process metrics
   - Collects **Prometheus metrics**
   - Supports **NVIDIA GPU monitoring** via DCGM
   - Supports multiple third-party apps (same as logging)

---

## 🛠 Real-World Use (with Your Project Experience)

> In production observability setups, we installed **Ops Agent** on all Compute Engine VMs using a **Terraform-based automation** pipeline.
> - The YAML configuration helped us filter and structure logs from custom Java applications.
> - We shipped Prometheus-formatted metrics and structured logs to GCP Logging and integrated them with custom alert policies.
> - Fluent Bit and OpenTelemetry support meant we didn’t need to maintain separate agents.
> - This significantly simplified troubleshooting and reduced agent overhead.

---

## 🎯 Interview Questions + Sample Answers

**Q1: Why should you prefer Ops Agent over legacy agents?**  
**A:** Ops Agent combines logging and monitoring in one lightweight agent, supports modern OSes, YAML-based config, and third-party application observability — unlike deprecated legacy agents.

---

**Q2: How does Ops Agent collect logs and metrics?**  
**A:** It uses Fluent Bit to collect logs and OpenTelemetry Collector for metrics and traces, which are forwarded to Cloud Logging and Monitoring respectively.

---

**Q3: How can you install Ops Agent across multiple VMs?**  
**A:** Options include using `gcloud`, Cloud Monitoring dashboard, or fleet-based tools like Ansible, Terraform, and Puppet.

---

**Q4: Can Ops Agent parse unstructured logs?**  
**A:** Yes. It supports regex and JSON parsing rules in YAML to convert text logs into structured formats.

---

**Q5: What third-party applications does Ops Agent support?**  
**A:** Kafka, Flink, HBase, MongoDB, MySQL, PostgreSQL, nginx, Apache, etc. are supported for both log and metric collection.

---

## 🧠 Analogy to Remember

🔌 **One Smart Plug Analogy**:
Before, you had two separate plugs — one for logging, one for monitoring. Ops Agent is a **smart plug** that does both jobs — efficiently, faster, and with better compatibility.

---

## 🔧 CLI Command Summary

```bash
# Install Ops Agent on a single VM (Linux)
curl -sSO https://dl.google.com/cloudagents/add-google-ops-agent-repo.sh
sudo bash add-google-ops-agent-repo.sh --also-install

# Restart Ops Agent
sudo service google-cloud-ops-agent restart

# YAML configuration location
sudo nano /etc/google-cloud-ops-agent/config.yaml

# Validate agent status
sudo service google-cloud-ops-agent status

# Enable Ops Agent during VM creation
gcloud compute instances create vm-with-ops-agent \
  --metadata=enable-oslogin=TRUE \
  --metadata=google-monitoring-enable=TRUE,google-logging-enable=TRUE
