## Phase 1
* Create a Regional GKE Cluster
* Create a namespace "production"
* Implement prod-quota 
    * Limiting to 10 CPU and 16Gi RAM max per namespace
    * Every container must explicitly declare resources.limits

## Phase 2
  * Utilizing Dynamic Provisioning via GKE's standard-rwo class
  * mango-pvc that triggers cloud to create a persistent disk
  * Deployed MangoDB to manage data, ensuring the database always finds its specific disk even if moved to a different node.
  * Note that even if we do not create a storage class for pvc a default is taken while in     admission

## Phase 3
  * Front-end uses init-containers. Once DB is reachable it pings the db services and allows app to start.
  * MangoDB (1 replica) Frontend(2 replicas)
  * Update all the yamls to include cpu and memory to satisfy phase 1 resource quota
