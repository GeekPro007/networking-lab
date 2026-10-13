# Multi-Cloud Networking Capstone Lab - Solutions & Troubleshooting Log



## INC-4521: API Service Outbound Connectivity Failure



### 1. Challenge & Symptoms

- **Problem Statement:** The internal API service running on the private subnet (10.0.2.2) was unable to fetch data or communicate with external public APIs over HTTPS.

- **Diagnostic Command:**

  ```bash

  curl -4 -fsS --max-time 10 https://api.ipify.org

  ```
---

## Incident Resolution Report: INC-4522

### 1. Incident Overview
* **Incident ID**: INC-4522
* **Environment**: Google Cloud Platform (`vpc-networking-lab-cf870758`)
* **Affected Nodes**:
  * API Server: `vm-api-cf870758` (`10.0.2.2`)
  * Database Server: `vm-database-cf870758` (`10.0.3.2`)
* **Status**: Resolved & Verified

---

### 2. Issue Description
The Flask API hosted on `vm-api-cf870758` was unable to communicate with the PostgreSQL database server (`db.internal.test`) on TCP port `5432`. Troubleshooting identified two independent root causes: private DNS resolution failure and an overly restrictive VPC firewall rule.

---

### 3. Root Cause & Remediation Steps

#### Step A: Private DNS Resolution
* **Problem**: Executing `dig @169.254.169.254 +short db.internal.test` returned `NXDOMAIN`.
* **Root Cause**: The Cloud DNS Managed Zone (`internal-test-cf870758`) was bound to the default network rather than the custom VPC network `vpc-networking-lab-cf870758`.
* **Fix**:
  ```bash
  gcloud dns managed-zones update internal-test-cf870758 \
- **Symptom:** Requests timed out with `curl: (28) Failed to connect to api.ipify.org port 443 after 8753 ms: Connection timed out`.



### 2. Diagnosis & Root Cause

- **Context:** The API server resides in a private subnet without a public IP and relies on Cloud NAT for internet egress.

- **Root Cause:** The Cloud NAT gateway (nat-cf870758) on the regional router (router-cf870758) was misconfigured and did not permit traffic from all VPC subnet ranges.



### 3. Resolution

Executed the following command from the local admin terminal:

```bash

gcloud compute routers nats update nat-cf870758 \

    --router=router-cf870758 \

    --region=us-central1 \

    --auto-allocate-nat-external-ips \

    --nat-all-subnet-ip-ranges

```



### 4. Verification & Evidence

- **Validation Command:**

  ```bash

  curl -4 -fsS --max-time 10 https://api.ipify.org

  ```

- **Result:** Successfully returned public IP `35.239.183.217`.



---



## INC-4522: Service Discovery & Private DNS Resolution



### 1. Challenge & Symptoms

- **Problem Statement:** Microservices could no longer resolve internal hostnames (`db.internal.test`, `web.internal.test`, `api.internal.test`).

- **Diagnostic Command:**

  ```bash

  dig +short db.internal.test

  ```

- **Symptom:** Returned empty output.



### 2. Diagnosis & Root Cause

- **Context:** Internal service discovery relies on Google Cloud DNS Private Managed Zones.

- **Root Cause:** The private DNS managed zone (`internal-test`) lacked an active visibility network binding with the lab's VPC network.



### 3. Resolution

Executed the following command from the local terminal:

```bash

gcloud dns managed-zones update internal-test \

    --networks=$(gcloud compute networks list --format="value(name)" | head -n 1)

```



### 4. Verification & Evidence

- **Validation Command:**

  ```bash

  dig +short db.internal.test

  ```

- **Result:** Successfully resolved to private IP `10.0.3.2`.
