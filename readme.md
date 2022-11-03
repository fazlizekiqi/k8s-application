### Fibbonaci Application

This project is an over-complicated fibonacci application where you can type in a number and see the calculated fibonacci number.
The reason for overcomplicating stuff is just to create a small prototype in order to test the power of Kubernetes.

The same methodology can be used to develop and deploy real-life application and use the power of Kubernetes to scale, automate and manage containerized applications. 

### Tools used in the application 
    - Docker 
    - Kubernetes  Version 1.22
    - Google Cloud Platform

### Services used in the application:
    - client (The web-platform. Developed with ReactJS)
    - server (The rest-api server that the client communitaces with. Developed with Node + ExpressJS)
    - worker (The worker that does the actual fibonacci calculation. Developed with Node)
    - postgresql (Saves the indexes/inputs. Developed with Node)
    - redis ( Saves the calculated fibonacci values. Developed with Node) 

### Secret object in Kubernetes

Securely stores a piece of information in the cluster, such as a database password or some data that we dont want to add it in the config files.
Since we usually sensitive data in the config files the most common way to create a secret is by using the imperative command in Kubernetes. 

To create a secret we run the following command:

    kubectl create secret generic <secret_name> --from-literal key=value

  - **create** ( Imperative command to create a new object. Can be used to created Pods, Deployments etc...)
  - **secret** ( Type of object we are going to create.)
  - **generic** ( Type of secret object. generic indicates tjat we are saving some arbitrary number of key-value pairs. There are 2 other types of secrets that we can make use of.
    1. *docker-registr*y (Useful anytime we want to setup some type of authentication with a custom docker registry that we can store our images in)
    2. _tls_ ( Related to HTTPS setup. Useful when storing TLS keys)
  - **<secret_name>** (Name of secret, for laster reference in a Pod config.)
  - **--from-literal** (We add the secret information into this command, as opposed to from . file so it does expect a key-value after)
  - **key=value** (Key-value pair of the secret information)


### Handling traffic with Ingress Controllers 

This project makes use of [Ingress-Nginx](https://github.com/kubernetes/ingress-nginx) and not the [Official Kubernetes Ingress Controller](https://github.com/nginxinc/kubernetes-ingress).

[Read more about the difference of them](https://grigorkh.medium.com/there-are-two-nginx-ingress-controllers-for-k8s-what-44c7b548e678)

**NOTE!**
Setup of ingress-nginx changes depending on the environment.
**In the local development we can simply apply one kubernetes config file and then run the application**

Application will use the same configuration file _ingres-service.yml_ for routing the traffic to our services in both environments:
 - For local development purposes
 - Google Cloud 

**NOTE! Only the ingress-nginx installation process can vary on different platforms.**

## Local Development 

#### While running the application locally we need to generate a secret with the postgres password since that is the only thing that this application is trying to hide from the outside world.
For local development do (first time only):

    kubectl create secret generic pgpassword --from-literal MYPGPASSWORD=123456

 - The 'pgpassword' will be used in the server-deployment(k8s/server-deployment.yml). That's why we need to generate the key first.
 - The same secret object will also be used in the (k8s/postgres-deployment.yml) in order to override the default password of Postgresql.
 It is recommended that we use env variables like this all the data that we think should not be exploited.

#### Setting up Ingress Controller Locally
 We can do the installation process with Helm or we can simply apply a kubernetes object with the below command: 
[More on installation guide for Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml

### Now run the application with the below commands:

    cd k8s-application
    kubectl apply -f k8s-dev

### Open browser and visit 
    http://localhost
    (Since the default port is 80 we dont need to specify one. But speificying one will also work http://localhost:80)

### As for now there are 3 replicas for the client-deployment.yml and server-deployment.yml
We can reduce the number of replicas by going inside those file changing them and then do a kubectl apply -f file.yml or
we can also reduce/scale it with the imperative commands:
```
    kubectl scale --replicas=1 -f client-deployment.yml
```
By accessing the:
```
    kubectl get deployments
```
we can see that the desired number of replicas is 1 in the client-deployment. 
### To stop all the kubernetes object running in our local development we do 
```
    kubectl delete -f .\k8s-dev\
```
and make sure that no Pod/Service object are runing by doing :
```
    kubectl get pods
    kubectl get services
```
 
### Accessing Kubernetes Dashboard Locally
To access the kubernetes dashboard make sure to follow the [the documentation in this file](k8s-dev/dashboard/kubernetes-dashboard.md)

### Production
At this point we need to go in the [Google Cloud Platform](https://console.cloud.google.com/) create a project and inside of that project we need to create kubernetes clusters.

By creating that project GCP will also provide us **project id** that is going to get used in the pipeline environment(github action).
Not only we need to create the clusters, but we also need a way to access the environment that those clusters
are running through our pipeline environment. In google cloud we can do that by creating a **Secret Account** and give it the necessary permissions which
will let us access the Google cloud environment. 


#### Production configuration
 For the production purposes we need to ssh into the google cloud environment and manually create our secret key for the postgresql just like we do for the local development. 
 In order to do that we need to go to: https://console.cloud.google.com/ and then press Activate Shell in the top right corner and configure the shell to access the right environment where our clusters upp and running.

If we do not see the pods when running the *kubectl get pods* and see the following error, that is because we need to configure google cloud to point to our project inside of that(since this doesnt come out of the box):
```
    The connection to the server localhost:8080 was refused - did you specify the right host or port?
```
So, that's why we need to configure the environment and that can be done by setting up the project id:
```
     gcloud config set project <project-id>
```
Settings up the zone:
```
    gcloud config set compute/zone <location> (Can be found under Kubernetes Cluster dashboard on the cluster that you have created eg. europe-north1-a under the location column)
```
Getting the credentials for the cluster that you create:
```
    gcloud container clusters get-credentials <cluster-name>
```
And now, after the configuration of gcloud inside of the google cloud environment we can create our secret by hitting the command:
```
     kubectl create secret generic pgpassword --from-literal MYPGPASSWORD=<your-secret-postgres-password>
```


Since for our local development we were using ingress-service.yml as an Ingress Controller, we kind of need to do the same thing for the Google Cloud Provider as well.
So, how do we install the ingress-controller in our production environment(GCP). 
Well, there are multiple ways to install the NGINX ingress controller but we will be using [Helm - The package manager for Kubernetes](https://helm.sh/):

[Check the documentation if you want to use different installation alternatives](https://kubernetes.github.io/ingress-nginx/deploy/)

### Steps to install ingress-controller with Helm in the Google Cloud

In your Google Cloud Console run the following command to install helm:
```
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
    chmod 700 get_helm.sh 
    ./get_helm.sh
```
Then we install the Ingress-controller:
```
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    helm install my-release ingress-nginx/ingress-nginx
```

### Setting up a Domain (note! this will cost ~10$)

1. Domain Purchase 
 - Go to [Google Domains](http://domains.google.com/)
 - Search for a Domain and add it to the cart.
 - If you want yearly renew of this domain keep the **Auto-renew is on**  otherwise disable it.
 - Checkout
 - (You might need to add some extra personal information together with the credit card information after you checkout. Just fill in the info.)

2. Setup the domain
 - Go inside your kubernetes services in the google cloud. -> Kubernetes Engine -> Services & Ingress
 - Find the ingress-controller services and copy the IP address that is under the "Endpoints" column(the one that we can access to visit our website). 
 - Go back to the domains.google.com and find your domain 
 - Click on the DNS on the left side
 - In the Custom records section we are going to create 2 different records
 - One is for the your-domain.com and the other is for www.your-domain.com
 - For the your-domain.com add:
    1. Hostname: (leave empty)
    2. Type: A
    3. TTL: 3600 (TTL — (Time-To-Live) How often a copy of the record stored in cache (local storage) must be updated (fetched from original storage) or discarded. Shorter TTLs mean records are fetched more often (access is slower, data is more current). Longer TTLs mean records are fetched from less often (access is faster, data is less current). The default value is 1 hour.)
    4.  Paste the IP addressed that you copied from the Services & Ingress in the google cloud platform. Make sure to remove the / and the http://. It should look something like this _35.228.187.128_
 - For the www.your-domain.com add a nother custom record:
    1. Hostname: www
    2. Type: CNAME
    3. TTL: 3600 
    4. your-domain.com 
### Adding the certificate in the project:
 - Access the google cloud shell:
 - Add the Jetstack Helm repository
```
    helm repo add jetstack https://charts.jetstack.io
```
 - Update your local Helm chart repository cache:
```
   helm repo update
```
 - Install the cert-manager Helm chart:
```
    helm install \
    cert-manager jetstack/cert-manager \
    --namespace cert-manager \
    --create-namespace \
    --version v1.8.0 \
    --set installCRDs=true
```
This will download a couple of files and install them inside of our Kubernetes cluster as a couple of different new objects.
We also need 2 different config files 
 - One for the Issuer (issuer.yml)
 - One for the Certificate (certificate.yml)
