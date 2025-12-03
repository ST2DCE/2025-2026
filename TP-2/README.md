# TP-2 - Deployment orchestation using Kubernetes #

* Manage, scale, and maintain containerized applications
* Deployment in different environments
* Scalability
* Package management

## Prerequisites strategies ##

Donwload and install Kind, a tool for running local Kubernetes clusters using Docker container “nodes”. It is available from [kind.sigs.k8s.io](https://kind.sigs.k8s.io/docs/user/quick-start#installing-from-release-binaries) and the `kubectl` command-line utility, [install-kubectl](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/)

## Part 1: Set up a local Kubernetes cluster ##

Run `kind create cluster --name devops` for creating a Kubernetes cluster.
The cluster will be created and a configuration file is automatically created that `kubectl` can use to connect to the Kubernetes cluster.
Kubernetes organizes all of its resources into namespaces by different projects or teams. You must create a namespace in which to do all the work required. 
To create a namespace `kubectl create namespace dev` and use it by default: `kubectl config set-context --current --namespace=dev`.

## Part 2: Pods, Deployments, and Services ##

In this part, you'll use `kubectl` to create Kubernetes pods, deployments, and services without getting into any of the details around these concepts.

1. Use `kubectl create deployment nginx --image=nginx --port=80` to create a deployment witch launch an Nginx pod. This may take a few minutes as it must pull the Nginx container image down from the Internet.

2. Run `kubectl get pod,deployment` to see that the previous step created a deployment object as well as a single pod (managed by the deployment).

3. Run `kubectl describe deployment nginx` to get more details on the deployment created by step 1.

4. Run `kubectl describe pod <pod-name>` to get more details on the pod created by step 1.

5. Use `kubectl create service nodeport nginx --tcp=80:80` to make this Nginx pod accessible outside the cluster. Run `kubectl port-forward service/nginx 8082:80` to access to the nginx service.

6. Use `kubectl get pod,deployment,service` to see the objects created by the previous two steps. You should see a single deployment, a single pod, and a single service.

7. Run `kubectl describe service nginx` to get more details on the pod created by step 1. Note the "Selector" line, which shows that this service is using the "app=nginx" label to note the pods that will be part of this service.

8. Scale Deployment Use Cases:

Run `kubectl scale deployment --replicas=N nginx`

* Handling Traffic / Performing Load Testing
* Maintaining High Availability
* Adjusting Horizontal Auto-Scaling
* Optimizing Resource Utilization

9. Delete the service and the deployment:

Use `kubectl delete service nginx` and `kubectl delete deployment nginx` respectively, and  `kubectl get pods` to verify that deleting the deployment also deleted the pod.

## Part 3: Manage our backend and frontend application ##

In this final part, you'll examine more details on pods, deployments, and services, and work with the YAML files that define these objects in Kubernetes.

To do this, we're going to start again with our application used in the previous workshop:

we're using a private registry on docker hub to store versions of our images, so we'll need to think about creating a secret so that we can retrieve built images.

Type the following command:

```console
 kubectl create secret docker-registry efrei2023 --docker-server=docker.io --docker-username=efrei2023 --docker-password=efrei2023 --docker-email=abdoulzak@proton.me
```

This time we'll assume we're in a production environment. Remember to create the namespace **prod** and use it by default.

### Backend deployment ###

1. Examine the `backend-deployment.yaml` file :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  labels:
    name: backend
spec:
  selector:
    matchLabels:
      app: backend
  replicas: 1
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend-container
          image: efrei2023/backend:$TAG
          imagePullPolicy: Always
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRES_PASSWORD
              value: postgres
            - name: POSTGRES_USER
              value: postgres
      imagePullSecrets:
        - name: efrei2023 
---
apiVersion: v1
kind: Service
metadata:
  name: backend
  labels:
    app: backend
spec:
  type: NodePort
  ports:
    - port: 5432
  selector:
    app: backend

```

2. Create the deployment by running `kubectl apply -f backend-deployment.yaml`. Use `kubectl get pods,deployments,services` to see that a deployment was created.

### Frontend deployment ###

Based on the backend configuration file, try writing the frontend configuration file with the following details.

For the Deployment:

* The image needs to be `efrei2023/frontend:$TAG`.
* The port exposed from the container should be **80**.
* Label the pods with `app: frontend`.

For the Service:

* The type of _Service_ should be `NodePort`.
* The service port should be **80**.
* The target port should be **8000**.
* Use the label `app` and the value `frontend` for the selector.

## Accessing and Using the App ##

Once the two YAMLs have been applied:

* Access your application with `kubectl port-forward service/frontend 8080:8000`.

## Kubernetes package manager : Helm ##

One of the many advantages of Helm is that it gives you a one-command installation method for many popular apps3 steps to have a fully deployed and rununig apps :

  Step 1 - Add the Repo : `helm repo add bitnami https://charts.bitnami.com/bitnami`

  Step 2 - Install the Helm Chart : 
  
  `helm repo update`
  
  `helm install my-blog bitnami/wordpress --set wordpressUsername=admin --set wordpressPassword=admin --set service.type=NodePort`

  Step 3 - Access your app: `kubectl port-forward service/my-blog-wordpress 8080:80`

### Kubernetes dashboard overview ###

The Dashboard is a web-based Kubernetes user interface. You can use it to:

* Deploy containerized applications to a Kubernetes cluster
* Troubleshoot your containerized application
* Manage the cluster resources
* Get an overview of applications running on your cluster
* Creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc)

Install the kubernetes Dashboard :

`helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/`

`helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard`

Configure Cluster Access with the following content dashbord-config.yaml :

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: dev
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: dev
```
Apply the configuration : `kubectl apply -f dashboard-config.yaml`

create an access token for the Dashboard with the command : `kubectl create token admin-user`

Access to the UI: `kubectl  port-forward service/kubernetes-dashboard-kong-proxy 8443:443`

## ArgoCD Overview ##

* Step 1 -  kubectl create namespace argocd

* Step 2 -  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

* Step 3 -  kubectl get all -n argocd

* Step 4 -  kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d 

* Step 5 -  kubectl port-forward svc/argocd-server -n argocd 8081:443

