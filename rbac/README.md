https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user
https://kubernetes.io/docs/tasks/administer-cluster/certificates/
https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication

# You can enable multiple authentication methods at once. You should usually use at least two methods:

* at least one other method for user authentication.
* service account tokens for service accounts

service account tokens for service accounts
at least one other method for user authentication.

## Authentication and Authorization with normal user

A few steps are required in order to get a normal user to be able to authenticate and invoke an API. First, this user must have a certificate issued by the Kubernetes cluster, and then present that certificate to the Kubernetes API.

### Create private key 

The following scripts show how to generate PKI private key and CSR. It is important to set CN and O attribute of the CSR. CN is the name of the user and O is the group that this user will belong to. You can refer to RBAC for standard groups.

```bash 
   openssl genrsa -out myuser.key 2048
   openssl req -new -key myuser.key -out myuser.csr
```

### Create CertificateSigningRequest 
Create a CertificateSigningRequest and submit it to a Kubernetes Cluster via kubectl. Below is a script to generate the CertificateSigningRequest.

```bash 
    kubeclt apply -f certificate-signing-request.yml
```
Some points to note in the certificate-signing-request.yml file :

* usages has to be 'client auth'
* expirationSeconds could be made longer (i.e. 864000 for ten days) or shorter (i.e. 3600 for one hour)
* request is the base64 encoded value of the CSR file content. You can get the content using this command: cat myuser.csr | base64 | tr -d "\n"

### Approve certificate signing request 

Use kubectl to create a CSR and approve it.
Get the list of CSRs:
```bash 
    kubectl get csr
```
Approve the CSR:

```bash 
    kubectl certificate approve bedrilcsr 
```

### Get the certificate

Retrieve the certificate from the CSR:

```bash 
    kubectl get csr/bedrilcsr  -o yaml 
```
The certificate value is in Base64-encoded format under status.certificate.

Export the issued certificate from the CertificateSigningRequest.


```bash 
    kubectl get csr bedrilcsr -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
```

### Create Role and RoleBinding
With the certificate created it is time to define the Role and RoleBinding for this user to access Kubernetes cluster resources.

This is a sample command to create a Role for this new user:

```bash 
    kubeclt apply -f role.yml
```

This is a sample command to create a RoleBinding for this new user:

```bash 
    kubeclt apply -f role-binding.yml
```

### Add to kubeconfig 

The last step is to add this user into the kubeconfig file.

First, you need to add new credentials:

```bash 
    kubectl config set-credentials bedril --client-key=myuser.key --client-certificate=myuser.crt --embed-certs=true
```

Then, you need to add the context:

```bash 
    kubectl config set-context bedril --cluster=kubernetes --user=bedril
```

To test it, change the context to bedril:

```bash 
    kubectl config use-context bedril
```

## Authentication and Authorization service account tokens

You can see :
* create-token-from-service-account and how-to-bind-role-toservice-account capture 
* cluster-autoscaler-autodiscover.yaml