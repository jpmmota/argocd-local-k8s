# ArgoCD

This repository describes how to install and test GitOps with ArgoCD in a local kubernetes cluster using Docker Desktop.

## ArgoCD Installation
**Create namespace and deploy ArgoCD**
```sh
kubectl create namespace argocd
kubectl apply -n argocd -f argocd-server-deployment.yaml
```
**Deploy ingress-nginx**
```sh
kubectl apply -n ingress-nginx-f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml
```

**Ingress configuration**
```sh
kubectl apply -n argocd -f ./ingress-nginx/deployment.yaml
```

**Get admin password**
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d > admin_pass.txt
```

**Open application**
- URL: http://localhost
- Credentials: admin / "Value from admin_pass.txt"

### ArgoCD CLI Installation
**Linux**
```sh
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.1.5/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

**MacOS (Homebrew)**
```sh
brew install argocd
```

**Authenticate CLI**
```sh
argocd login localhost --insecure
```
- Credentials: admin / "Value returned from Get admin password command"

## Create new application and sync

**Fork example application**
Fork [ArgoCD example repository](https://github.com/codefresh-contrib/gitops-certification-examples/tree/main/simple-app)

Repository URL:
[https://github.com/jpmmota/argocd-example](https://github.com/jpmmota/argocd-example)


**Create ArgoCD application and Sync**
```sh
kubectl create namespace webapp

argocd app create webapp \
--project default \
--repo https://github.com/jpmmota/argocd-example \
--path "./simple-app" \
--dest-namespace webapp \
--dest-server https://kubernetes.default.svc \
--grpc-web

argocd app sync webapp
```

**Port-forward to test the application in the local browser**
```sh
kubectl port-forward -n webapp service/simple-service 31000:8080
```

## Reconciliation

Bump the container version in the deployment.yml (line 18) as shown below:
*from*:
```yaml
image: docker.io/kostiscodefresh/gitops-simple-app:v1.0
```

*to*:
```yaml
image: docker.io/kostiscodefresh/gitops-simple-app:v2.0
```

Sync the application:
```sh
argocd app sync webapp
argocd app wait webapp
````
