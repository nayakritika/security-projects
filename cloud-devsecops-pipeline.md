# Cloud DevSecOps Pipeline — AWS Infrastructure as Code

**TL;DR:** Provisioned AWS infrastructure using Terraform (EC2 instances, security groups), managed containerised workloads with Docker/nerdctl, and set up a Kubernetes cluster from scratch — covering the full IaC and container orchestration stack.

## Context

Lab-based exploration of Infrastructure as Code on AWS and Kubernetes, building practical skills applied directly in the larger K8s hardening project. Covered Terraform provisioning, container runtime management, CNI selection, and worker node joining via Ansible.

Course: Large Systems, University of Amsterdam (OS3)

## What Was Covered

**Terraform on AWS:**
- Provisioned EC2 instances and security groups
- Used `terraform plan` / `apply` / `output` workflow
- Verified HTTP connectivity between instances

```bash
# Terraform created: 1 Security Group + 2 EC2 Instances
curl http://54.234.110.14   # "This page is served by instance number 1."
curl http://54.91.139.128   # "This page is served by instance number 2."
```

**Container runtime:**
- Ran and managed containers with nerdctl/containerd
- Tested lifecycle management and HTTP endpoint verification

**Kubernetes cluster setup:**
- Installed kubeadm v1.30, kubelet, kubectl
- Configured Calico CNI for pod networking and NetworkPolicy enforcement
- Joined worker nodes via Ansible playbooks
- Resolved UFW/iptables conflicts with cluster networking

## Calico vs Flannel

Calico builds a real Layer-3 routed network using BGP — better performance, scalability, and built-in NetworkPolicy enforcement. Flannel uses VXLAN overlay — simpler, sufficient for small clusters. Calico is the right choice for any production or security-sensitive environment.

## Tools & Technologies

`Terraform` `AWS EC2` `Docker` `Kubernetes v1.30` `Calico` `Flannel` `Ansible` `kubeadm` `containerd`

---

**Ritika Nayak** — MSc Security & Network Engineering, University of Amsterdam (OS3)
*Lab work, Large Systems course*
