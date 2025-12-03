# TP-3 - Monistoring and Incidents Management #

* Monitor the application / platform

 - Install and configure Prometheus / Grafana / AlertManager stack
 - Install node_exporter and view metrics on Grafana
 - Confifure alerts, collect and process logs

The main goal of this workshop is to deploy Prometheus and Grafana into your Kubernetes cluster using Helm charts. 
Prometheus is used to monitor Kubernetes Cluster and other resources running on it. Globaly it is a systems and service monitoring tool.
Grafana helps to visualize metrics recorded by Prometheus and display them in dashboards.

All installation is done in the 'monitoring' namespace. 

## Install prometheus using Official Helm Chart ##

Step 1 - Create the namespace : `kubectl create namespace monitoring`

Step 2 - Add prometheus repository : 

 ```shell
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts`
```

Step 2 - Install provided Helm chart for Prometheus : 
 
 ```shell
helm install prometheus prometheus-community/prometheus --namespace monitoring
 ```
Step 3 - Access Prometheus UI : 

 ```shell
kubectl --namespace=monitoring port-forward deploy/prometheus-server 9090
 ```

## Repeat steps 1 - 4 for the Grafana component using Official Helm Chart ##

* Grafana Helm repo :  https://grafana.github.io/helm-charts

* Install helm Chart 'grafana/grafana' and  get your 'admin' user password.

* Access Grafana UI :

 ```shell
 kubectl --namespace=monitoring port-forward deploy/grafana 3000
```

* Access Grafana Web UI and configure a datasource with the deployed prometheus service url. 

* Install and Explore Node_Exporter Dashborad. ID 1860

## Configure AlertManager component ##

* Get the Alertmanager URL by running these commands in the same shell :

  ```shell
  export POD_NAME=$(kubectl get pods --namespace monitoring -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace monitoring port-forward $POD_NAME 9093
  ```
