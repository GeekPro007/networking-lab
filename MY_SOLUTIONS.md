## INC-4521: API Service Can't Pull External Data

### Issue
The API server residing in the private subnet was unable to initiate outbound connections to external internet endpoints (e.g., timing out on requests to https://example.com and https://api.ipify.org).

### Root Cause
The existing Cloud NAT gateway (nat-8183452c) on Cloud Router router-8183452c was configured with sourceSubnetworkIpRangesToNat: LIST_OF_SUBNETWORKS and was only routing traffic for the database subnet (subnet-database-8183452c). The API server's subnet was excluded.

### Resolution
Updated the Cloud NAT configuration to include all subnet IP ranges:

```bash
gcloud compute routers nats update nat-8183452c \n    --router=router-8183452c \n    --region=us-central1 \n    --nat-all-subnet-ip-ranges
```

### Verification
Ran diagnostic curl requests from the API server:
- curl -4 --noproxy '*' -sS -o /dev/null -w '%{http_code}\n' --max-time 10 https://example.com (Returned HTTP 200)
- curl -4 --noproxy '*' -fsS --max-time 10 https://api.ipify.org (Returned egress IP 34.41.17.0)

## INC-4522: Service Discovery & Private DNS Resolution Failure

### Issue
Internal service discovery failed when resolving internal hostnames within the VPC. Instances could not resolve private DNS endpoints.

### Root Cause
The Private DNS Managed Zone was not properly bound to the VPC network (`vpc-networking-lab-8183452c`), preventing internal DNS queries from resolving private domain entries.

### Resolution
Associated the Private DNS Managed Zone with the target VPC network:
```bash
gcloud dns managed-zones update <ZONE_NAME> \
    --networks=vpc-networking-lab-8183452c
