---
title: Google Cloud Key Management Service
description: Learn about Google Cloud Key Management Service
---

## Step-01: Introduction
- Understand about Encryption Types
    - Symmetric Key Encryption
    - Asymmetric Key Encryption
- Understand about Cloud KMS   

## Step-02: Create Key Ring
- Go to Security -> Data Protection -> Key Management -> 
- Click on **CREATE KEY RING**
- **Key Ring Name:** my-keyring1
- **Location type:** Region (Lower latency within single region)
- **Region:** us-central1
- Click on **CREATE**

## Step-03: Create Key
- Go to Security -> Key Management -> my-keyring1
- Click on **CREATE KEY**
### Name and protection level
- **Key Name:** my-symkey-1
- **Protection Level:** software
### key material
- **Generated Key:** checked
### Purpose and Algorithm
- **Purpose:** Symmetric encrypt/decrypt
- **Algorithm:** Google symmetric key (leave to default)
### Versions
- **Key Rotation period:** 90 days (leave to default)
- **Starting on:** leave to default
### Additional Settings
- **Duration of 'scheduled for destruction' state:** 5 (default 30 days)
- REST ALL LEAVE TO DEFAULTS
- Click on **CREATE**

## Step-04: Review the newly created Symmetric Key
- Go to Security -> Key Management -> my-keyring1 -> my-symkey-1
- Review options like
    - Disable
    - Destroy

## Step-05: Review Key creation for Assymetric Encryption
- Assymetric Sign
- Assymetric Decrypt    


## Step-06: gcloud: Create KMS Key ring and Keys
```t
# Create KMS Keyring - Regional
gcloud kms keyrings create my-keyring2 --location us-central1

# Create KMS Keyring - Global
gcloud kms keyrings create my-keyring3 --location global

# Create a symmetric encryption key with custom automatic rotation 
gcloud kms keys create KEY_NAME \
    --keyring KEY_RING \
    --location LOCATION \
    --purpose "encryption" \
    --protection-level "software" \
    --destroy-scheduled-duration SCHEDULED_FOR_PERMANENT_DESTRUCTION_AFTER_DAYS

# Replace Values
gcloud kms keys create my-symkey-2 \
    --keyring my-keyring2 \
    --location us-central1 \
    --purpose "encryption" \
    --protection-level "software" \
    --destroy-scheduled-duration "2d"    

# List Keys
gcloud kms keys list --keyring my-keyring2 --location us-central1

# Describe Key
gcloud kms keys describe my-symkey-2 --keyring my-keyring2 --location us-central1
```

## Step-07: Clean-Up
```t
# Destroy my-symkey-2
- Go to Security -> Key Management Service -> my-keyring2 -> my-symkey-2 -> Destroy all key version material
- Click on "SCHEDULE DESTRUCTION"

# my-symkey-1
We will use this "my-symkey-1" from my-keyring1 in next demo, so we will not destroy it
```

---
title: Google Cloud Key Management Service (KMS) – Overview
description: Understand KMS concepts, encryption types, key hierarchy, and integration with GCP services. Learn when to use symmetric vs asymmetric encryption.
---

## ✅ Step-by-Step Summary

### 1. Understand Data States
Google classifies data into three states — all of which need encryption:
- **Data at rest**: Stored on disks, backups, archives
- **Data in transit**: Moving within or outside Google Cloud (e.g., App ↔ DB)
- **Data in use**: Actively processed in RAM/CPU

---

### 2. Types of Encryption

#### 🔐 Symmetric Encryption
- One key is used for **both** encryption and decryption.
- Fast, lightweight, and suitable for **bulk transfers**.
- Examples: AES, DES, 3DES, IDEA

##### ✔ Advantages
- High performance
- Lower CPU/memory usage
- Industry standard (widely adopted)

##### ❌ Disadvantages
- Key sharing can be risky
- You must securely transmit the encryption key

---

#### 🔐 Asymmetric Encryption
- Two keys: **Public Key** (encrypt) and **Private Key** (decrypt)
- Slower, more resource-intensive
- Ideal when **key distribution** is a concern

##### ✔ Advantages
- Public key can be shared; private key stays secret
- Stronger protection for sensitive key material

##### ❌ Disadvantages
- Slower than symmetric
- Not suitable for large or high-throughput data transfers

---

### 3. Google Cloud Key Management Service (KMS)

KMS enables you to create, use, rotate, and manage cryptographic keys securely and centrally.

- Supports both **symmetric** and **asymmetric** encryption
- Provides **APIs for encryption, decryption, and digital signing**
- Can be used with:
  - Compute Engine (boot disks)
  - Cloud Storage
  - Cloud SQL
  - Your own applications

---

### 4. Key Management Options in GCP

| Option                             | Description |
|----------------------------------|-------------|
| **Google-managed encryption**     | Default, no user setup required |
| **Customer-managed encryption**   | Keys managed via GCP KMS |
| **Customer-supplied encryption**  | Keys stored and managed outside of Google Cloud |

---

### 5. Viewing Encryption Options on a VM
When creating a VM:
1. Go to **Compute Engine → Create Instance**
2. In **Boot Disk → Change → Show Advanced Options**
3. View encryption choices:
   - Google-managed key (default)
   - Customer-managed key (KMS)
   - Customer-supplied key (external)

---

## 🛠 Real-World Use (with Your Project Experience)

> During a secure data pipeline implementation, we used **Customer-Managed Encryption Keys (CMEK)** to encrypt sensitive backups in Cloud Storage and boot disks in Compute Engine.
>
> We used **KMS-symmetric keys** with auto-rotation every 90 days and controlled key access using IAM roles.
> Applications interfaced with the KMS API to encrypt user-submitted documents before persistence.
>
> This provided centralized control, auditability, and compliance with internal security policies.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is the difference between symmetric and asymmetric encryption?**  
**A:** Symmetric uses the same key to encrypt/decrypt. Asymmetric uses a public/private key pair. Symmetric is faster and suitable for bulk data; asymmetric is more secure for key exchange.

---

**Q2: What are the 3 data states in GCP security?**  
**A:** Data at rest (storage), data in transit (network), and data in use (in memory/CPU). GCP secures all three using encryption strategies.

---

**Q3: When should you use customer-managed keys (CMEK)?**  
**A:** When you want control over encryption lifecycle, key rotation, IAM access, and audit logs for compliance or policy enforcement.

---

**Q4: Does Google Cloud encrypt all data by default?**  
**A:** Yes. All data is encrypted at rest using **Google-managed keys** by default. Users can optionally supply CMEK or CSEK.

---

## 🧠 Analogy to Remember

🔑 **House Lock Analogy**:
- **Symmetric encryption**: One key opens and locks the door — you must share it securely.
- **Asymmetric encryption**: Anyone can lock your door (public key), but only you can unlock it (private key).
- **KMS**: Your key vault with programmable access control, audit logging, and remote control.

---

## 🔧 CLI Command Summary (Preview)

```bash
# List key rings
gcloud kms keyrings list --location=global

# Create key ring
gcloud kms keyrings create my-key-ring --location=global

# Create symmetric key
gcloud kms keys create my-sym-key \
  --location=global \
  --keyring=my-key-ring \
  --purpose=encryption

# Encrypt a file
gcloud kms encrypt \
  --location=global \
  --keyring=my-key-ring \
  --key=my-sym-key \
  --plaintext-file=secret.txt \
  --ciphertext-file=secret.enc

# Decrypt a file
gcloud kms decrypt \
  --location=global \
  --keyring=my-key-ring \
  --key=my-sym-key \
  --ciphertext-file=secret.enc \
  --plaintext-file=secret_decrypted.txt
```



````md
---
## Title: Google Cloud KMS – Create and Use Symmetric Keys
## Description: Learn how to create symmetric encryption keys using Google Cloud Key Management Service (KMS) via Console and CLI, and use them to encrypt Compute Engine resources.
---

## ✅ Step-by-Step Summary

### Step 1: Create a Key Ring via Console
- Navigate to: `Security > Key Management > Create Key Ring`
- **Name**: `my-key-ring-1`
- **Location Type**: Regional
- **Region**: `us-central1`
- Click **Create**

---

### Step 2: Create a Symmetric Key in the Key Ring
- After creating the key ring, click **Create Key**
- **Key Name**: `my-symmetric-key-1`
- **Protection Level**: Software
- **Key Material**: Generated key (default)
- **Purpose**: Symmetric Encrypt/Decrypt
- **Rotation Period**: 90 days
- **Destruction Grace Period**: 5 days
- Click **Create**

---

### Step 3: Use KMS Key During VM Creation
- Go to `Compute Engine > Create VM`
- Under **Boot Disk > Change > Show Advanced Options**
- Choose **Customer-managed encryption key (CMEK)**
- Select `my-key-ring-1 > my-symmetric-key-1`

---

### Step 4: Create Key Ring and Key via gcloud CLI

```bash
# Set your GCP project
gcloud config set project YOUR_PROJECT_ID

# Create a regional key ring
gcloud kms keyrings create my-key-ring-2 --location=us-central1

# Create a symmetric key with rotation and destruction policy
gcloud kms keys create my-symmetric-key-2 \
  --location=us-central1 \
  --keyring=my-key-ring-2 \
  --purpose=encryption \
  --protection-level=software \
  --rotation-period=90d \
  --next-rotation-time=$(date -u +"%Y-%m-%dT%H:%M:%SZ" -d "+90 days") \
  --destroy-scheduled-duration=2d
````

---

### Step 5: View, List, and Describe Keys

```bash
# List keys in a key ring
gcloud kms keys list \
  --location=us-central1 \
  --keyring=my-key-ring-2

# Describe a specific key
gcloud kms keys describe my-symmetric-key-2 \
  --location=us-central1 \
  --keyring=my-key-ring-2
```

---

### Step 6: Destroy a Key Version (Manual Clean-Up)

```bash
# Destroy key version via Console:
# Navigate to Key > Select version > Destroy Key Material

# Example via gcloud:
gcloud kms keys versions destroy 1 \
  --location=us-central1 \
  --keyring=my-key-ring-2 \
  --key=my-symmetric-key-2
```

---

## 🛠 Real-World Use (with Your Project Experience)

We encrypted boot disks of VM instances using **CMEK (Customer-Managed Encryption Keys)** to meet internal security and compliance requirements.
Keys were managed with **auto-rotation every 90 days**, access-controlled through IAM, and monitored using audit logs.
This setup allowed secure provisioning of Compute Engine VMs with customized encryption policies, especially beneficial for storing sensitive application data.

---

## 🎯 Interview Questions + Sample Answers

**Q1: What is the difference between Google-managed and customer-managed encryption keys?**
**A:** Google-managed keys are used by default with no manual setup. Customer-managed keys (CMEK) are created and managed by the user, providing control over rotation, access, and audit.

---

**Q2: What is a key ring in GCP KMS?**
**A:** A key ring is a container for organizing cryptographic keys in a specific location. It groups related keys and simplifies key lifecycle management.

---

**Q3: Can a CMEK key be rotated automatically?**
**A:** Yes. You can set a rotation period (e.g., every 90 days) at key creation. GCP will automatically rotate the key as scheduled.

---

**Q4: Can KMS be used outside of Google Cloud services?**
**A:** Yes. KMS provides APIs for encrypting, decrypting, and signing data, allowing use in custom applications hosted in GCP or externally.

---

## 🧠 Analogy to Remember

🔐 **Key Management as a Bank Vault**

* **Google-managed key** = Free bank locker; you don’t manage the lock.
* **Customer-managed key** = Your personal vault with programmable access, audits, and rotating locks.
* **Customer-supplied key** = You bring your own lock and key, and the bank never sees it.

---

## 🔧 CLI Command Summary

```bash
# Create Key Ring
gcloud kms keyrings create my-key-ring-1 --location=us-central1

# Create Symmetric Key
gcloud kms keys create my-symmetric-key-1 \
  --location=us-central1 \
  --keyring=my-key-ring-1 \
  --purpose=encryption \
  --protection-level=software

# List Keys
gcloud kms keys list --location=us-central1 --keyring=my-key-ring-1

# Describe Key
gcloud kms keys describe my-symmetric-key-1 \
  --location=us-central1 \
  --keyring=my-key-ring-1

# Destroy Key Version
gcloud kms keys versions destroy 1 \
  --location=us-central1 \
  --keyring=my-key-ring-1 \
  --key=my-symmetric-key-1
```

---

## 📚 Additional References

* [Cloud KMS Overview](https://cloud.google.com/kms/docs)
* [Create and Manage Keys](https://cloud.google.com/kms/docs/creating-keys)
* [CMEK with Compute Engine](https://cloud.google.com/compute/docs/disks/customer-managed-encryption)
* [Key Rotation Best Practices](https://cloud.google.com/kms/docs/key-rotation)

```

Let me know when you’re ready to proceed to the next section on **Compute Engine Disks with CMEK integration**!
```


