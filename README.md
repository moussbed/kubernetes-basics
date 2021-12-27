### The basics of Kubernetes

#### Start minikube 
```shell
    $ minikube start --driver=docker
```

#### Encode mongo user and mongo password 
```shell
    $ echo -n mongouser | base64
    $ echo -n mongopassword | base64
```

#### Install Ingress Controller in Minikube

https://kubernetes.io/docs/concepts/services-networking/ingress/
https://kubernetes.github.io/ingress-nginx/deploy/

```shell
     $ minikube addons enable ingress
```  

#### Launch deployments,secrets, config map and Ingress
```shell
   $ kubectl apply -f mongo-secret.yaml 
   $ kubectl apply -f mongo-config.yaml
   $ kubectl apply -f mongo.yaml 
   $ kubectl apply -f webapp.yaml
   $ kubectl apply -f webapp-ingress.yaml
```   

#### TO verify 

```shell
    $ curl $(minikube ip): $(webapp-service port: find in the command kubectl get service)

```

