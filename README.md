# K8s-Django-App

In this project, we will create a Django web application and deploy it on Kubernetes using Minikube. This process involves the creation of deployment files and services. To handle environment variables, we'll employ Kubernetes secrets. Additionally, for load balancing and port forwarding, we will utilize ingress-nginx.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Check The Connectivity](#check-the-connectivity)
- [Detailed Information](#detailed-information)

## Prerequisites

Ensure that you have the following prerequisites installed on your system:

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)

## Getting Started

1. Clone this repository to your local machine:

```shell
git clone https://github.com/Harasisco/K8s-Django-App.git
cd K8s-Django-App
```

2. Start Minikube from a terminal with administrator access (but not logged in as root):

```shell
minikube start
```

3. Enable the NGINX Ingress controller:

```shell
minikube addons enable ingress
```

**Verify that the NGINX Ingress controller is running:**

```shell
kubectl get pods -n ingress-nginx
```

4. Create our custom namespaces:

```shell
kubectl create namespace development
kubectl create namespace database
```

5. Deploy the MySQL database:
   - Start with our service and secret environment file:

   ```shell
   kubectl create -f mysql-service.yaml
   kubectl create -f mysql-secret.yaml
   ```

   - Then deploy the MySQL database:
     
   ```shell
   kubectl create -f mysql-deploy.yaml
   ```

   - Check if the MySQL database started correctly:
     
   ```shell
   kubectl get pods,svc -n database
   kubectl logs pod/<your mysql pod> -n database
   ```

6. Deploy the Django app:
   - Start with our service and secret environment file:
     
   ```shell
   kubectl create -f web_app_secret.yaml
   kubectl create -f web_app_service.yaml
   ```

   - Create our ingress-nginx Port Forwarder:
     
   ```shell
   kubectl create -f web_app_ingress.yaml
   ```
   
   - Deploy the Django app:
   
   **Before deploying the web app, make sure everything before is working correctly**
     
   ```shell
   kubectl create -f web_app_deploy.yaml
   ```

7. Verify that the Ingress controller is directing traffic:

```shell
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info\n
```

<p>For additional insight into your cluster state, minikube bundles the Kubernetes Dashboard, allowing you to get easily acclimated to your new environment: </p>

**make sure to run this command in new tab**

```shell
minikube dashboard
```
## Check The Connectivity
- Firstly execute the MySQL container using:
```shell
kubectl exec -it <full POD name> -n database -- /bin/bash
```
- Secondery check the MySQL data base info:
```shell
$ mysql -p
> show databases;
> use my_database;
> show tables
```
- You will see that No tables shown.
 
- In a new tab execute the Django container same as what we did with the MySQL container, then:
 ```shell
kubectl exec -it <full POD name> -n development -- /bin/bash
cd mysite/
python manage.py migrate
```
- Finally, Go back to the MySQL tab and rerun the ` show tables; ` command to figoure out that the Django spp and MySQL database are linked correctly.

## Detailed information

### Why using Kubernetes Secrets
<p>Kubernetes secrets serve as a secure vault for confidential data like passwords and API keys that applications need to operate.
They offer a protective layer against exposing sensitive information directly in code or configuration files, reducing the risk of unauthorized access. 
By storing secrets separately in Kubernetes, we enhance security, ensuring that only authorized entities can access and manage this critical information. 
This separation of concerns not only fortifies the application against potential security breaches but also simplifies the management of sensitive data in a scalable and efficient manner.</p>

### Why using Ingress-NGINX
<p>Ingress is an API object that defines rules for how external traffic should be routed to services. 
The NGINX Ingress Controller interprets these Ingress rules and configures the NGINX server accordingly.</p>
