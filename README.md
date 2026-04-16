# Kubernetes Lab 4: Storage, Metadata & Configuration

This repository contains the practical implementation of **Kubernetes Lab 4**, focusing on **Persistent Storage**, **Dynamic Metadata**, and **Decoupled Configurations**.

---

## 📁 Task 1: Persistent Volumes & Deployments
**Objective:** Implement a reliable storage layer using Persistent Volumes (PV) and Claims (PVC) to serve static content via Nginx replicas, ensuring the data survives pod restarts and is accessible across specific nodes.

### 1. Host Node Preparation
To simulate a persistent backend, we prepared the host node by creating a physical directory and the initial data file.

* **Action:** Created directory `/mnt/data` on the host.
* **Content:** Generated `index.html` containing the full name "Omar Alaa El-Din".
* **Permissions:** Applied `chmod 644` to ensure the web server has appropriate read access.

**📸 Verification:**

![Host Setup](Screenshot%202026-04-14%20221308.png)

---

### 2. Storage Infrastructure (PV & PVC)
We utilized a Manual Storage Class to bind a specific Persistent Volume to a Claim, ensuring dedicated storage for our deployment.

* **Persistent Volume (PV):** Named `nginx-pv` with `1Gi` capacity and `ReadWriteMany` access mode.
* **Reclaim Policy:** Set to `Retain` to prevent data loss if the PVC is deleted.
* **Binding:** Successfully bound the `nginx-pvc` to the `nginx-pv`.

**📸 Manifest & Binding:**

![PV PVC Manifest](Screenshot%202026-04-15%20223041.png)

![Binding Status](Screenshot%202026-04-15%20223040.png)

---

### 3. Nginx Deployment & Node Affinity
The deployment was configured to serve the mounted data with high availability across a specific target node.

* **Scaling:** Configured with **3 replicas**.
* **Node Scheduling:** Used `nodeName: ubuntu` to force pods onto the node where the `hostPath` exists.
* **Volume Mounting:** Mounted the `nginx-pvc` to `/usr/share/nginx/html` inside the containers.

**📸 Deployment Verification:**

![Deployment Manifest](Screenshot%202026-04-15%20225918.png)

![Running Pods](Screenshot%202026-04-15%20225859.png)

---

### 4. Final Solution Validation
To verify the end-to-end flow, we accessed the running pods to confirm the persistent data was being served correctly.

* **Test:** Executed `cat` on the `index.html` file from inside one of the replicas.
* **Result:** The pod successfully displayed the name "Omar Alaa El-Din" retrieved from the persistent host storage.

**📸 Output Verification:**

![Final Result](Screenshot%202026-04-15%20230433.png)

---

## 📁 Task 2: Metadata Injection via Downward API
**Objective:** Expose dynamic Pod information (Name and IP) to the container without making direct calls to the Kubernetes API server.

### 1. Persistent Storage for Metadata
To meet the lab requirements, a dedicated Persistent Volume and Claim were created to store and serve the injected metadata.

* **PV & PVC:** Defined `downward-pv` and `downward-pvc` with `manual` storage class to ensure a stable binding.
* **Status:** Both resources are successfully **Bound**, as shown in the cluster resource list.

**📸 Verification:**

![Downward PV PVC Binding](Screenshot%202026-04-16%20001521.png)

---

### 2. Downward API Deployment Logic
The deployment was configured with a hybrid approach to overcome the limitations of mounting dynamic fields like Pod IP as volume files in certain environments.

* **Environment Variables:** Used to fetch the dynamic `status.podIP`.
* **Downward API Volume:** Used to mount the `metadata.name` as a file.
* **Service Automation:** A shell script was embedded in the container command to aggregate this data into a custom `index.html`.

**📸 Manifest Details:**

![Downward API YAML 1](Screenshot%202026-04-16%20002022.png)

![Downward API YAML 2](Screenshot%202026-04-16%20002055.png)

---

### 3. Verification of Injected Metadata
The success of this task was confirmed by checking the live container output, which serves the Pod's own identity data.

* **Pod Status:** Verified that `downward-deployment` is running successfully.
* **Final Output:** Accessing the pod revealed the correct Pod Name and Pod IP displayed in the serve file.

**📸 Execution Result:**

![Downward API Verification](Screenshot%202026-04-16%20001632.png)

![Downward API Final Output](Screenshot%202026-04-16%20002501.png)

---

## 📁 Task 3: Configuration Management (ConfigMaps)
**Goal:** Decouple application configuration from the container image using ConfigMaps for environment variables and volume mounts.

### 1. Creating ConfigMaps (YAML vs. Imperative)
Two ConfigMaps were created to demonstrate different management styles in Kubernetes.

* **ConfigMap `birke`:** Created via a YAML manifest (`/opt/cm.yaml`) containing structured data (tree, level, department).
* **ConfigMap `trauerweide`:** Created using the imperative CLI command with literal values.

**📸 Creation Verification:**

![ConfigMap YAML Creation](Screenshot%202026-04-16%20190441.png)

![Imperative ConfigMap Creation](Screenshot%202026-04-16%20190518.png)

---

### 2. Pod Configuration (`pod1`)
A dedicated Pod was deployed to consume these ConfigMaps using two distinct methods:

* **Environment Variable:** Injected the key `tree` from ConfigMap `trauerweide` as `TREE1`.
* **Volume Mount:** Mounted the entire `birke` ConfigMap as a set of files under `/etc/birke`.

**📸 Pod Manifest:**

![Pod1 ConfigMap YAML](Screenshot%202026-04-16%20190702.png)

---

### 3. Final Lab Validation
The implementation was verified by inspecting the internal state of `pod1`.

* **Env Test:** `printenv TREE1` successfully returned `trauerweide`.
* **Volume Test:** `ls /etc/birke` confirmed that all data keys appeared as individual files.
* **Content Test:** `cat /etc/birke/tree` correctly displayed the value `birke`.

**📸 Verification Commands:**

![ConfigMap Final Validation](Screenshot%202026-04-16%20190901.png)
