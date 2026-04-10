This folder contains the Kubernetes manifests for Secure-MediaVault, a multi-tier application deployed on Google Kubernetes Engine (GKE). The project demonstrates advanced Kubernetes concepts including Namespace Isolation, Resource Guardrails, Stateful Persistence, and Ordered Service Discovery.

**Deployment Order & Logic**

The manifests are designed to be applied in a specific sequence. This ensures that the infrastructure "laws" (Quotas) and the "foundation" (Storage) are active before the applications are deployed.

1. *The Environment & Law (01-namespace.yaml, 02-vault-quota.yaml)
Isolation*: Everything is contained in the media-vault namespace.

The Bouncer (ResourceQuota): We enforce a strict "Hard Ceiling" of 4 CPU Cores and 8Gi RAM. This prevents costs from spiraling and ensures one buggy pod cannot crash the entire cluster.

Mandatory Limits: Because a Quota is active, every single container in this project must specify resources.limits or GKE will reject the deployment.

2. *The Persistent Foundation (redis-setup.yaml - PVC Section)
Storage Strategy*: We use Dynamic Provisioning via the standard-rwo (ReadWriteOnce) StorageClass.

Data Integrity: A 5Gi Google Persistent Disk is requested. This disk lives outside the lifecycle of the pods. If the Redis pod crashes or the node is upgraded, the disk remains, and the data is preserved.

3. *The Stateful Backend (redis-setup.yaml - Deployment & Service)
Deployment*: Runs Redis 6-Alpine. It is configured with exactly 1 replica to prevent data corruption and disk-locking (since a standard GKE disk can only be "owned" by one pod at a time).

Volume Mapping:

PVC: Requests the 5GB disk from Google.

Volume: Plugs that physical disk into the Pod.

VolumeMount: Maps the container's /data folder to that disk.

Internal Service: A ClusterIP service named redis-service creates a stable internal DNS entry for the frontend to find the database.

4. *The Resilient Frontend (gallery-ui.yaml)
High Availability*: Deployed with 3 Replicas to ensure the UI remains online even if a node fails.

Init-Containers (Pre-flight Check): The check-redis init-container uses nslookup to ping redis-service. The main Gallery application will not start until the database is verified as "Online" and "Searchable" via DNS.
