# For demo we are using GKE, create admin user role binding for installing prometheus cluster roles

kubectl create clusterrolebinding clusteradmin-gcpaccount --clusterrole=cluster-admin --user=<>@gmail.com

# Create Namespace for logging

kubectl create namespace monitoring

# Create ServiceAccount for Helm

kubectl apply -f helm/helm.yml

#Init Helm

helm init --service-account=helm

# Install Ingress controller to Access Kibana/Prometheus/Grafana from Public.

helm install  --name my-nginx-ingress stable/nginx-ingress --set rbac.create=true  --set controller.metrics.serviceMonitor.enabled=true

# Deploy Prometheus to K8s cluster

kubectl apply -f prometheus/ -n monitoring

# Deploy Grafana to K8s cluster

kubectl apply -f grafana/ -n monitoring

# Deploy node-exporter to K8s cluster

kubectl apply -f node-exporter/ -n monitoring

# Deploy kube-state-metrics to K8s cluster

kubectl apply -f kube-state-metrics/ -n monitoring

# Deploy ingress object for grafana/prometheus public access to K8s cluster

kubectl apply -f ingress.yaml

# Access the Logs from Grafana UI using Ingress Controller Public IP as DNS record,

<ingress controller IP> A grafana.cloudnloud.in
