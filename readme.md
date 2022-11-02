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
Setup of ingress-nginx changes depending on the environment(i.e configuration files changes depending on which cloud provider we are deploying to).

Application have 2 different configuration files:
 - For local development purposes
 - Google Cloud 

## Local Development 

#### While running the applciation locally we need to generate a secret with the postgres password since that is the only thing that this application is trying to hide from the outside world.
For local development do (first time only):

    kubectl create secret generic pgpassword --from-literal PGPASSWORD=123456

 - The 'pgpassword' will be used in the server-deployment(k8s/server-deployment.yml). That's why we need to generate the key first.
 - The same secret object will also be used in the (k8s/postgres-deployment.yml) in order to override the default password of Postgresql.
 It is recommended that we use env variables like this all the data that we think should not be exploited.

#### Setting up Ingress Controller Locally
 
 - **Make sure to remove the ingress.yml file if the command below is working for you..**

[More on installation guide for Nginx Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/#quick-start)

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.4.0/deploy/static/provider/cloud/deploy.yaml

### Now run the application with the below commands:

    cd k8s-application
    kubectl apply -f k8s

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
    kubectl delete -f .\k8s\
```
and make sure that no Pod/Service object are runing by doing :
```
    kubectl get pods
    kubectl get services
```
 
### Accessing Kubernetes Dashboard Locally
To access the kubernetes dashboard make sure to follow the [the documentation in this file](k8s/dashboard/kubernetes-dashboard.md)
