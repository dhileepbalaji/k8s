# For demo we are using GKE, disable default logging in GKE to stack driver.

gcloud container clusters update standard-cluster-1 --zone us-central1-a  --logging-service none

kubectl create clusterrolebinding clusteradmin-gcpaccount --clusterrole=cluster-admin --user=<>@gmail.com

# Create Namespace for logging

kubectl create namespace logging

# Create ServiceAccount for Helm

kubectl apply -f helm/helm.yml

#Init Helm

helm init --service-account=helm

# Install Ingress controller to Access Kibana/Prometheus/Grafana from Public.

helm install  --name my-nginx-ingress stable/nginx-ingress --set rbac.create=true  --set controller.metrics.serviceMonitor.enabled=true


# Deploy ElasticSearch Using helm charts

helm install --name elasticsearch stable/elasticsearch --set master.persistence.enabled=true --set data.persistence.enabled=true --set image.tag=6.4.2 --set cluster.name=elasticsearch-test --set master.replicas=3 --set client.ingress.enabled=false --namespace logging

# Deploy Kibana Using helm charts

helm install --name kibana stable/kibana --set env.ELASTICSEARCH_URL=http://elasticsearch-client:9200 --set image.tag=6.4.2 --set ingress.enabled=true --set ingress.hosts[0]=kibana.cloudnloud.in  --set ingress.annotations[0]="kubernetes.io/ingress.class: nginx" --namespace logging

# Deploy Fluentbit Daemonset to K8s cluster

kubectl apply -f fluentbit/ServiceAccount.yml

kubectl apply -f fluentbit/ClusterRole.yml

kubectl apply -f fluentbit/ClusterRoleBinding.yml

kubectl apply -f fluentbit/fluent-bit-config.yml

kubectl apply -f fluentbit/fluent-bit-ds.yml

# Deploy Fluentd Deployment to K8s cluster

helm install fluentd/ --name fluentd --set output.host=elasticsearch-client --set persis
tence.enabled=true --set replicaCount=1 --set service.externalPort=24224 --set service.ports[0].containerPort=24224 --set output.port=9200 --namespace logging

# Run Test Nginx container to log messages:

kubectl run nginx --image=nginx -n logging

kubectl port-forward <nginxpodname> 8081:80 -n logging &

while true; do curl localhost:8081; sleep 2; done

# Access the Logs from Kibana UI using Ingress Controller Public IP as DNS record,

<ingress controller IP> A kibana.cloudnloud.in
