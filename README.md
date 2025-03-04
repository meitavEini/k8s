# Kubernetes WordPress Deployment

## Table of Contents
1. [Project Files](#project-files)
2. [AWS EKS Deployment](#aws-eks-deployment)
   1. [Prerequisites](#prerequisites)
   2. [Update Kubeconfig](#update-kubeconfig)
   3. [Create/Use a Namespace](#createuse-a-namespace)
   4. [Apply the Manifests](#apply-the-manifests)
   5. [Potential Issues](#potential-issues)
3. [AWS ECR (Optional): Push Images](#aws-ecr-optional-push-images)
4. [Troubleshooting](#troubleshooting)

---

## 1. Project Files

This repository contains YAML manifests required to deploy WordPress and MariaDB on Kubernetes:

1. **mariadb-statefulset.yaml**  
   - Defines a StatefulSet for MariaDB.  
   - Uses environment variables for database credentials.  
   - Mounts a persistent volume for `/var/lib/mysql`.  

2. **mariadb-service.yaml**  
   - Exposes MariaDB internally in the cluster on port 3306.  

3. **wordpress-deployment.yaml**  
   - Defines a Deployment for WordPress.  
   - Configured with environment variables for DB connection.  
   - Uses port 80 for HTTP.  

4. **wordpress-service.yaml**  
   - Exposes WordPress as a LoadBalancer service.  

5. **ingress.yaml**  
   - Defines an Ingress resource for external access.  

6. **ingressclass.yaml** (Optional)  
   - Defines an IngressClass if needed.  

7. **update_kubeconfig.sh** (Optional)  
   - Updates kubeconfig for EKS cluster access.  

---

## 2. AWS EKS Deployment

### 2.1 Prerequisites
- AWS CLI installed and configured.
- Kubernetes CLI (`kubectl`) installed.
- Helm installed.
- A running Amazon EKS cluster.

### 2.2 Update Kubeconfig
```sh
./update_kubeconfig.sh
kubectl get nodes
```

### 2.3 Create/Use a Namespace
```sh
kubectl create namespace ns-meitavi
```

### 2.4 Apply the Manifests
```sh
kubectl apply -f mariadb-statefulset.yaml -n ns-meitavi
kubectl apply -f mariadb-service.yaml -n ns-meitavi
kubectl apply -f wordpress-deployment.yaml -n ns-meitavi
kubectl apply -f wordpress-service.yaml -n ns-meitavi
kubectl apply -f ingress.yaml -n ns-meitavi
kubectl apply -f ingressclass.yaml -n ns-meitavi  # Optional
```

After applying, verify the resources:
```sh
kubectl get all -n ns-meitavi
kubectl get ingress -n ns-meitavi
```

---

## 3. AWS ECR (Optional): Push Images

If you need to store custom-built images in AWS ECR:
```sh
aws ecr create-repository --repository-name my-wordpress
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

Tag and push images:
```sh
docker tag wordpress:latest <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-wordpress:latest
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-wordpress:latest
```

---

## 4. Troubleshooting

### 4.1 Ingress Not Accessible
Ensure the Ingress Controller is running:
```sh
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```
If missing, install it:
```sh
helm install meitav-ingress ingress-nginx/ingress-nginx \
  --namespace ns-meitavi \
  --set controller.ingressClassResource.name=meitavi-ingress-class \
  --set controller.service.type=LoadBalancer
```

### 4.2 Persistent Volume Claims (PVC) Issues
Check PVC status:
```sh
kubectl get pvc -n ns-meitavi
kubectl describe pvc <pvc-name> -n ns-meitavi
```
If stuck in `Pending`, check the associated StorageClass.

### 4.3 Webhook Validation Issues
If you get webhook-related errors:
```sh
kubectl delete ValidatingWebhookConfiguration ingress-nginx-admission
```

---

Now your WordPress deployment should be up and running on EKS!

