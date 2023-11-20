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
**You will see something like thie:**
```shell
NAME                                        READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-dm92w        0/1     Completed   0          66s
ingress-nginx-admission-patch-zmqwz         0/1     Completed   1          66s
ingress-nginx-controller-7c6974c4d8-z9xfj   1/1     Running     0          66s
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
curl --resolve "hello-world.info:80:$( minikube ip )" -i http://hello-world.info
```

**You will get something like this:**

```html
<h1>Hello, World!</h1>
```

<p>For additional insight into your cluster state, minikube bundles the Kubernetes Dashboard, allowing you to get easily acclimated to your new environment: </p>

**make sure to run this command in new tab**

```shell
minikube dashboard
```
## Check The Connectivity
- Firstly execute the MySQL pod using:
```shell
kubectl exec -it <full POD name> -n database -- /bin/bash
```
- Secondery check the MySQL data base info:
```shell
$ mysql -p
> show databases;
> use my_database;
> show tables;
```
- You will see that No tables shown.
 
- In a new tab execute the Django pod same as what we did with the MySQL pod, then:
 ```shell
kubectl exec -it <full POD name> -n development -- /bin/bash
cd mysite/
python manage.py migrate
```
**You will get this:**
```shell
$ kubectl exec -it django-app-deploy-7df69494dd-tn72r -n development -- /bin/bash
root@django-app-deploy-7df69494dd-tn72r:/code# cd mysite/
root@django-app-deploy-7df69494dd-tn72r:/code/mysite# python manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying auth.0009_alter_user_last_name_max_length... OK
  Applying auth.0010_alter_group_name_max_length... OK
  Applying auth.0011_update_proxy_permissions... OK
  Applying auth.0012_alter_user_first_name_max_length... OK
  Applying sessions.0001_initial... OK
root@django-app-deploy-7df69494dd-tn72r:/code/mysite# 
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
