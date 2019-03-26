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

kubectl apply -f prometheus/




# Deploy Fluentd Deployment to K8s cluster

helm install fluentd/ --name fluentd --set output.host=elasticsearch-client --set persis
tence.enabled=true --set replicaCount=1 --set service.externalPort=24224 --set service.ports[0].containerPort=24224 --set output.port=9200 --namespace logging

# Run Test Nginx container to log messages:

kubectl run nginx --image=nginx -n logging

kubectl port-forward <nginxpodname> 8081:80 -n logging &

while true; do curl localhost:8081; sleep 2; done

# Access the Logs from Kibana UI using Ingress Controller Public IP as DNS record,

<ingress controller IP> A kibana.cloudnloud.in
