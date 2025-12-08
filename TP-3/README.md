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
* Analyze the content below, update the default prometheus configuration to set up an alert:

```yaml
serverFiles:
  alerting_rules.yml:
    groups:
      - name: Instances
        rules:
          - alert: InstanceDown
            expr: up == 0
            for: 1m
            labels:
              severity: critical
            annotations:
              description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
              summary: "Instance {{ $labels.instance }} down"
```

Apply the configuration update with : `helm upgrade --reuse-values -f prometheus-alerts-rules.yaml prometheus prometheus-community/prometheus`

Delete the 'prometheus-prometheus-pushgateway' deployment and see that you have an alert in AlertManager UI.

## Configure AlertManager to send Alerts by Email ##

Create the file *alertmanager-config.yaml* with the configuration and apply it with the following command ( Official email_config : https://prometheus.io/docs/alerting/latest/configuration/#email_config):

```console
   helm upgrade --reuse-values -f alertmanager-config.yaml prometheus prometheus-community/prometheus
```

## Configure Grafana/Loki ##

Loki is Grafana’s log aggregation component. Inspired by Prometheus, it’s highly scalable and capable of handling petabytes of log data.

Install the component by using the  Helm chart provided by grafana :

Create a file loki-config.yaml with the following contents:

```yaml
auth_enabled: false

server:
  http_listen_port: 3100

loki:
  commonConfig:
    replication_factor: 1
  useTestSchema: true
  storage:
    bucketNames:
      chunks: chunks
      ruler: ruler
      admin: admin
deploymentMode: SingleBinary
singleBinary:
  replicas: 1
read:
  replicas: 0
write:
  replicas: 0
backend:
  replicas: 0
```

... then install with the command:

```console
helm install loki grafana/loki -f loki-config.yaml -n monitoring
```

Expose Loki service  in order to access UI with target port 3100 : `kubectl port-forward --namespace monitoring svc/loki-gateway 3100:80`

To continue the exercise, follow the instructions displayed on the console after installation.

## Install  Grafana/Loki Agent (Alloy) ##

Install Alloy  Helm chart with the provided config:

```console
helm install alloy grafana/alloy  -f alloy-config.yaml -n monitoring
```

Configure Loki datasource on Grafana UI and run queries.