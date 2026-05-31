# Kubernetes Security Hardening — HA Cluster on Bare Metal

**TL;DR:** Designed and deployed a production-grade highly available Kubernetes cluster across 6 VMs on 3 physical servers using Terraform and Ansible. Validated under Locust load testing — zero failures up to 4,600 concurrent users, <1% failure rate at 6,000.

## Context

This project combined infrastructure as code, configuration management, and container orchestration to create a reproducible, fault-tolerant deployment. The goal was to compare orchestration-level scaling (replica counts) versus infrastructure-level scaling (adding worker nodes) under real load.

Course: Large Systems, University of Amsterdam (OS3)

## Architecture

```
kubectl → Load Balancer 1 (HAProxy/Keepalived) ─┐
          Load Balancer 2 (HAProxy/Keepalived) ─┤
                                                 ├─ Control Plane 1
                                                 ├─ Control Plane 2
                                                 ├─ Control Plane 3
                                                 ├─ Worker Node 1
                                                 ├─ Worker Node 2
                                                 └─ Worker Node 3
                                                         │
                                                      MetalLB → External Traffic
```

3 control-plane nodes + 3–7 worker nodes across 3 physical servers. No single point of failure at any layer.

## Implementation

**Infrastructure provisioning (Terraform + cloud-init):**
- 6 VMs provisioned across 3 physical servers, 2 vCPUs / 2GB RAM each
- cloud-init bootstrapped all nodes into Kubernetes-ready state on first boot
- Remote GitLab backend for Terraform state — enables collaboration and consistent management

**Cluster automation (Ansible — fully idempotent):**
- Configured HAProxy + Keepalived on load balancers for VIP
- Installed containerd runtime on all nodes
- Ran `kubeadm init` on primary control plane, joined remaining nodes via generated tokens
- Applied Calico CNI for pod networking and NetworkPolicy enforcement
- Deployed MetalLB for bare-metal load balancing with static external IP assignment

**Application:** Google Online Boutique (microservices demo, 11 services)

## Load Testing Results (Locust)

| Test | Result |
|---|---|
| Zero failures threshold | 4,600 concurrent users |
| <1% failure rate | 6,000 concurrent users |
| <10% failure rate | 10,000 concurrent users |

**Key finding:** Orchestration-level scaling (more replicas) improved latency under moderate load but hit node resource limits. Infrastructure-level scaling (adding worker nodes) provided better scalability at high load by reducing resource contention and enabling more effective pod scheduling.

## Tools & Technologies

`Kubernetes v1.30` `Terraform` `Ansible` `Calico CNI` `MetalLB` `HAProxy` `Keepalived` `Locust` `containerd` `kubeadm`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Group project, Large Systems course*
