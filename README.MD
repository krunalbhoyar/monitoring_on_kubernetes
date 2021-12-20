# Monitoring: Prometheus and Grafana

setup is on EC@ instance on which minikube is installed
In security group all TCP ports is active

follow this article: everything is given here
https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/ 


Procedure:
1) create namespace "monitoring" 
$ kubectl create ns monitoring

# ++++++Setting up Prometheus+++++++
Reference: https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/

go inside 'kubernetes-prometheus' folder and create all resources
A) Create clusterRole
$ kubectl create -f clusterRole.yaml

B) Create configMap
$ kubectl create -f config-map.yaml

C) Create a Prometheus Deployment
$ kubectl create  -f prometheus-deployment.yaml 

D) Create service in monitoring namespace
$ kubectl create -f prometheus-service.yaml --namespace=monitoring

Dont create ingress here

$ kubectl get all --namespace=monitoring

Browse it: http://<your-node-ip>:30000   node-ip is the EC2 instance ip (in security group all TCP ports is active)


# ++++Setting Up Kube State Metrics++++
Reference: https://devopscube.com/setup-kube-state-metrics/

go inside "kube-state-metrics-configs" folder
Create all resources
$ kubectl apply -f .

$ kubectl get deployments kube-state-metrics -n kube-system
All the above kube state metrics objects will be deployed in the kube-system namespace


# ++++Setting Up Grafana++++
Reference: https://devopscube.com/setup-grafana-kubernetes/

go inside "kubernetes-grafana" folder

A) Create data source configuration
$ kubectl create -f grafana-datasource-config.yaml

B) Create ConfigMap to paas grafana.ini file to grafana container
$ kubectl create configmap grafana-config --from-file=grafana.ini -n monitoring

C) Create Deployment
$ kubectl create -f deployment.yaml

D) Create service
$ kubectl create -f service.yaml

Browse it: http://<your-node-ip>:32000         (in security group all TCP ports is active)
User: admin
Pass: admin


# ++++++Setting Up Node Exporter+++++++
Reference: https://devopscube.com/node-exporter-kubernetes/
go inside "kubernetes-node-exporter" folder

A) Create daemonset
$ kubectl create -f daemonset.yaml

B) Create service 
$ kubectl create -f service.yaml
___________________________________________________________________

Installation part is over now its time to access dashboards

You can change the namespace with command
$ kubectl config set-context --current --namespace=monitoring
$ kubectl config view


To access dashboards browse node ip:32000 (grafana) , ip:30000 (prometheus) make sure in security group (all tcp connection should activate)

+++In another way you can access:+++

A) access prometheus:
port forword:                                           (deployment name)
$ kubectl --namespace=monitoring port-forward deploy/prometheus-deployment 9090
You will get ip and browse it at 127.0.0.1:9090 u will get prometheus dastboard

Note:- it dont work if you access you k8s cluster via aws cli
  
B) access grafana  :                       (Pod_Name)
kubectl port-forward -n monitoring grafana-6cb5cf45bf-p2zjf 3000

>> grafana Import:\
8588 - Kubernetes Deployment Statefulset Daemonset metrics\
1860 - Node Exporter Full\
9964 - Jenkins: Performance and Health Overview\
805 - Node Exporter Server Metrics\
3119 - k8s cluster monitoring (via prometheus)
===============================================================================================================================

# +++++++++Theory+++++++++

>>Prometheus:

>>Kube state metrics:
you need to install "Kube state metrics" to get all the metrics related to kubernetes objects.
All the above kube state metrics objects will be deployed in the kube-system namespace
Kube state metrics service will provide many metrics which is not available by default. Please make sure you deploy Kube state metrics to monitor all your kubernetes API objects like deployments, pods, jobs, cronjobs etc..

>>Node exporter: 
 is an official Prometheus exporter for capturing all the Linux system-related metrics.
It collects all the hardware and Operating System level metrics that are exposed by the kernel.
You can use the node exporter to collect the system metrics from all your Linux systems

>>Grafana:

>>also set alert manager
https://devopscube.com/alert-manager-kubernetes-guide/

>>here deploy application as a nodeport service, loadvalancer is not supported in custom cluster
browse instance ip:30007  (30007 is a nodeport u assigned)

Other link:
setting of Prometheus and grafana:
https://www.youtube.com/watch?v=G5ar-evGJzE&t=1433s
