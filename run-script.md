# CLO835 Assignment 2 — Run Instructions (EC2 + kind)

This guide rebuilds the local single-node Kubernetes cluster (kind) and deploys:
- MySQL (ClusterIP service)
- Employees web app (NodePort 30000)

## Prerequisites (EC2)
Installed / available:
- docker
- kind
- kubectl
- git
- AWS CLI (optional; only needed if you want to pull from ECR)

Repository contains:
- manifests/00-namespaces/
- manifests/03-deployments/
- manifests/04-services/
- manifests/kind-config.yaml

Images used (example):
- 007259615064.dkr.ecr.us-east-1.amazonaws.com/mysql:v1
- 007259615064.dkr.ecr.us-east-1.amazonaws.com/employees:v2


---

## 0) Go to repo
```bash
cd ~/CLO835-Assignment2
git pull
```
1) Create kind cluster (NodePort 30000 mapped to EC2 host)

Delete old cluster if it exists:
```bash
kind delete cluster --name clo835 || true
```
Create new cluster using the config:
```bash
kind create cluster --name clo835 --config manifests/kind-config.yaml
```

Verify cluster health:
```bash
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl cluster-info --context kind-clo835
```

Report note (Q1): K8s API server IP
```bash
kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'; echo
```
2) Load container images into kind
Because this is a local kind cluster, Kubernetes pulls images from the kind node.
If ECR auth is not available on the EC2 instance, load images into kind.
Pull images (if needed):
```bash
docker pull 007259615064.dkr.ecr.us-east-1.amazonaws.com/mysql:v1
docker pull 007259615064.dkr.ecr.us-east-1.amazonaws.com/employees:v2
```

Load into kind:
```bash
kind load docker-image 007259615064.dkr.ecr.us-east-1.amazonaws.com/mysql:v1 --name clo835
kind load docker-image 007259615064.dkr.ecr.us-east-1.amazonaws.com/employees:v2 --name clo835
```

3) Create namespaces
```bash
kubectl apply -f manifests/00-namespaces
kubectl get ns
```

5) Create MySQL Service first (DNS name used by web app)
```bash
kubectl apply -f manifests/04-services/mysql-svc.yaml
kubectl get svc -n mysql
```

MySQL service DNS used by web app:
mysql-service.mysql.svc.cluster.local

5) Deploy MySQL and Web Deployments
Apply deployments:
```bash
kubectl apply -f manifests/03-deployments
kubectl get deploy -A
kubectl get pods -A -w
```

Wait until
mysql-deploy is 1/1 or 3/3 Running (depending on your final choice)
web-deploy is 3/3 Running


Important (recommended): MySQL replicas
MySQL should be 1 replica for consistent data (no built-in replication in this lab image):
```bash
kubectl scale deployment -n mysql mysql-deploy --replicas=1
kubectl get deploy -n mysql mysql-deploy
kubectl get pods -n mysql
```

6) Create Web NodePort Service (30000)
```bash
kubectl apply -f manifests/04-services/web-svc-nodeport.yaml
kubectl get svc -n web
kubectl get endpoints -n web web-service -o wide
```
Expected:
web-service shows NodePort 30000
endpoints show 3 pod IPs on port 8080

7) Test from EC2 (NodePort)
```bash
curl -i http://127.0.0.1:30000
```

8) Test from your laptop browser (external access)
Ensure EC2 Security Group inbound rule allows:
Custom TCP
Port 30000
Source: your laptop public IP (My IP)
Open in browser:
```bash
http://<EC2_PUBLIC_IP>:30000
```

10) Verify DB insert + get employee (optional evidence)
Get the current mysql pod name:
```bash
kubectl get pods -n mysql
```
Run a quick DB query:
```bash
kubectl exec -it -n mysql $(kubectl get pod -n mysql -l app=mysql -o jsonpath='{.items[0].metadata.name}') -- \
mysql -uadmin -padmin123 employees -e "SELECT * FROM employee ORDER BY emp_id DESC LIMIT 10;"
```
