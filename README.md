# Argo CD Playground

## Local Cluster with Kind and Ingress Controller

Install [kind](https://kind.sigs.k8s.io/docs/user/quick-start/)

```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## Bootstraping by Hand

### Install ArgoCD

```sh
kubectl create namespace argocd
kubectl apply -k bootstrap/argo-cd -n argocd
```

### Port Forward

Skip this step if you use ingress.

```sh
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Login via CLI

Fetch the initial password. The username is `admin`:

```sh
argocd login localhost:8080 --grpc-web --insecure --username admin --password $(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)
```

### Configure the repository

First create a personal access token on Github: https://github.com/settings/tokens. Use this token
as the password. You can skip the access token part if the repository is publicly accessible.

```sh
argocd repo add https://github.com/tom-schoener/argocd-playground.git --username tom-schoener --password <TOKEN>
```

### Self-Manage ArgoCD

```sh
kubectl apply -k bootstrap/self-manage -n argocd
```
