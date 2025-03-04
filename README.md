Kubernetes Project

Overview

This project includes Kubernetes configurations for deploying and managing an application using Helm, Ingress, and Prometheus monitoring.

Prerequisites

Kubernetes cluster

Helm installed

Git installed

Installation

1. Clone the Repository

git clone <repository-url>
cd <repository-folder>

2. Deploy NGINX Ingress Controller

helm install meitav-ingress ingress-nginx/ingress-nginx \
  --namespace ns-meitavi \
  --set controller.ingressClassResource.name=meitavi-ingress-class \
  --set controller.service.type=LoadBalancer

3. Deploy the Application

kubectl apply -f wordpress-ingress.yaml

4. Install Prometheus Stack

helm install meitavoush prometheus-community/kube-prometheus-stack --namespace ns-meitavi

Managing the Application

Check Running Services and Pods

kubectl get svc -n ns-meitavi
kubectl get pods -n ns-meitavi

Delete Persistent Volume Claims (PVC)

If a PVC is stuck in Terminating status:

kubectl delete pvc <pvc-name> --force --grace-period=0 -n ns-meitavi

Check Installed Helm Releases

helm list -n ns-meitavi

Troubleshooting

Ingress Issues

If you experience issues with Ingress validation:

kubectl delete ValidatingWebhookConfiguration ingress-nginx-admission

Helm Installation Fails

If an installation fails due to an existing release:

helm uninstall <release-name> -n ns-meitavi

Then retry the installation.

PVC Stuck in Terminating State

kubectl get pvc -n ns-meitavi
kubectl patch pvc <pvc-name> -p '{"metadata":{"finalizers":null}}' -n ns-meitavi

Git Configuration

Ignore Unwanted Files

Ensure .gitignore is configured correctly. Example:

*.log
*.tmp
node_modules/
.kube/
.vscode/

Commit Changes

git add .
git commit -m "Initial commit"
git push origin main

License

This project is open-source and free to use.
