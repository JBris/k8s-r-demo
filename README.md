# Kubernetes R demo

Quick demonstration of using R with Kubernetes

# Table of contents

- [Kubernetes R demo](#kubernetes-r-demo)
- [Table of contents](#table-of-contents)
- [Installing kind](#installing-kind)
- [Installing helm](#installing-helm)
- [Adding NGINX Ingress Operator](#adding-nginx-ingress-operator)
- [Adding CertManager](#adding-certmanager)
- [Creating Namespace](#creating-namespace)
- [Create pod](#create-pod)

# Installing kind

Install Docker first. Then...

```
# For AMD64 / x86_64
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-amd64
# For ARM64
[ $(uname -m) = aarch64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.26.0/kind-linux-arm64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

To create a cluster:

```
kind create cluster --name kind
```

To delete a cluster:

```
kind delete cluster --name kind
```

List clusters:

```
kind get clusters
```

# Installing helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
rm get_helm.sh
```

# Adding NGINX Ingress Operator

Install NGINX using helm

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

If you want a full list of values that you can set, while installing with Helm, then run:

```
helm show values ingress-nginx --repo https://kubernetes.github.io/ingress-nginx
```

To check which ports are used by your installation of ingress-nginx, look at the output of:

```
kubectl -n ingress-nginx get pod -o yaml
```

Get pods in ingress-inginx namespace:

```
kubectl get pods --namespace=ingress-nginx
```

Let's create a simple web server and the associated service:

```
kubectl create deployment demo --image=httpd --port=80
kubectl expose deployment demo
```

Then create an ingress resource. The following example uses a host that maps to localhost:

```
kubectl create ingress demo-localhost --class=nginx \
  --rule="demo.localdev.me/*=demo:80"
```

Now, forward a local port to the ingress controller:

```
kubectl port-forward --namespace=ingress-nginx service/ingress-nginx-controller 8080:80
```

Access localhost:8080

```
curl --resolve demo.localdev.me:8080:127.0.0.1 http://demo.localdev.me:8080
```

# Adding CertManager

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.1.1/cert-manager.yaml
```

# Creating Namespace

Create a namespace for the R pipelines workshop.

```
kubectl create ns r-pipelines-workshop
kubectl get ns
```

# Create pod

```
kubectl apply -f deployment/k8s
kubectl get pods -n r-pipelines-workshop
kubectl port-forward -n r-pipelines-workshop service/r-pipelines 9001 9000 5432 3838
```

Make several HTTP requests to the pod containers via curl

```
curl --resolve shiny.r-pipelines.nzsa.co.nz:3838:127.0.0.1 http://shiny.r-pipelines.nzsa.co.nz:3838

curl --resolve minio-api.r-pipelines.nzsa.co.nz:9000:127.0.0.1 http://minio-api.r-pipelines.nzsa.co.nz:9000

curl --resolve minio.r-pipelines.nzsa.co.nz:9001:127.0.0.1 http://minio.r-pipelines.nzsa.co.nz:9001
```

From the web browser

```
xdg-open http://127.0.0.1:3838
```