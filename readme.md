# CLO835 Assignment 2 — Deploy 2-Tier Application on Local Kubernetes (kind)

## Overview
This project deploys a containerized 2-tier application (Web + MySQL) on a locally simulated single-node Kubernetes cluster created using **kind** on an Amazon Linux EC2 instance.

The application consists of:
- Web application (employees app)
- MySQL database

Both components are deployed using Kubernetes:
- Pods
- ReplicaSets
- Deployments
- Services (NodePort and ClusterIP)

---

## Architecture
- Web Tier → exposed externally via **NodePort (30000)**
- MySQL Tier → internal only via **ClusterIP**
- Each tier runs in its own namespace

Namespaces:
- `web`
- `mysql`

---

## Prerequisites

EC2 Amazon Linux instance with:
- Docker
- kubectl
- kind
- Git
- Access to container images (ECR)

Open EC2 Security Group:
- SSH (22)
- TCP 30000 (for NodePort access)

---

## Clone Repository

```bash
git clone https://github.com/roshanstha1996/CLO835-Assignment2.git
cd CLO835-Assignment2
