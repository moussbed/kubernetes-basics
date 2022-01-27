We have two ways to create secret for private registry

### 1-way
This solution is useful when we want to index a large number of private registries
#### Login to private registry
Run this command inside the cluster

```bash
 $ aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin XXXXXXX.dkr.ecr.us-east-2.amazonaws.com 
```
This command create config.json in the .docker directory which contains the 
token which will be use to push and pull image from private registry.

#### Config this config.json file in the host where we run kubectl command

```bash
  $ scp -i $(minikube ssh-key) docker@$(minikube ip):.docker/config.json .docker/config-minikube.json # we rename to config-minikube.json because we already have this same on our host
```

#### Encoding content of config-minikube.json

```bash
  $ cat .docker/config-minikube.json | base64
```
Assign the command result to .dockerconfigjson in the docker-secret.yaml

#### Create secret

```bash
   $ kubectl create secret generic my-registry-key \
   > --from-file=.dockerconfigjson=.docker/config-minikube.json \
   > --type=kubernetes.io/dockerconfigjson
```
### 2-way
We use only one command to do it. But we index only one private registy

```bash
   $ kubectl create secret docker-registry my-registry-key-two \
   > --docker-server=https://XXXXXX.dkr.ecr.us-east-2.amazonaws.com \
   > --docker-username=AWS \ 
   > --docker-password=$(aws ecr get-login-password --region us-east-2)
```

### Use This Secret on our deployment
```yaml
    imagePullSecrets:
    - name: my-registry-key
```





