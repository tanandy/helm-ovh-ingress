# Deploy ingress controllers on OVH through Helm 3

## Pre-Requisites
* Create a Kubernetes Cluster
* Install Helm 3 (https://helm.sh/docs/intro/install)


## Setup App

Create namespace `echo` for test purposes
```bash
kubectl create ns echo
```

Deploy hello app to the cluster
```bash
kubectl -n echo create deployment echo-app --image=mendhak/http-https-echo
```

Deploy ClusterIP service for the application
```
kubectl -n echo expose deployment echo-app --port 80 --target-port 80
```

## Setup Ingress Controllers

### Nginx

Install Nginx ingress controller
```
kubectl create ns ingress-nginx
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install -n ingress-nginx nginx ingress-nginx/ingress-nginx -f ingress/nginx/values.yaml
```

Use proxy protocol with limited ranges for security (optional)
```
# Retrieve egress-ips for OVH
kubectl get svc nginx-ingress-nginx-controller -n ingress-nginx -o jsonpath="{.metadata.annotations.lb\.k8s\.ovh\.net/egress-ips}"
# Update `values-upgrades-ips.yaml` with ovh ranges ips
vim values-upgrades-ips.yaml
# Upgrade traefik with dynamics retrieved values
helm upgrade --reuse-values -n ingress-nginx nginx ingress-nginx/ingress-nginx -f ingress/nginx/values-upgrade-ips.yaml
```

Deploy Ingress
```
kubectl -n echo apply -f ingress/nginx/ingress.yaml
```

### Traefik 2

Install Traefik 2 ingress controller
```
kubectl create ns ingress-traefik
helm repo add traefik https://helm.traefik.io/traefik
helm repo update
helm install -n ingress-traefik traefik traefik/traefik -f ingress/traefik/values.yaml
```

Use proxy protocol with limited ranges for security (optional)
```
# Retrieve egress-ips for OVH
kubectl get svc traefik -n ingress-traefik -o jsonpath="{.metadata.annotations.lb\.k8s\.ovh\.net/egress-ips}"
# Update `values-upgrades-ips.yaml` with ovh ranges ips
vim values-upgrades-ips.yaml
# Upgrade traefik with dynamics retrieved values
helm upgrade --reuse-values -n ingress-traefik traefik traefik/traefik -f ingress/traefik/values-upgrade-ips.yaml
```

Deploy IngressRoute
```
kubectl -n echo apply -f ingress/traefik/ingress.yaml
```

current result:

```
{
  "path": "/",
  "headers": {
    "host": "ip-xx-xx-xx-xx.sbg.lb.ovh.net",
    "x-request-id": "xxxxxxxxxxxxxxxxxxxxxxxxx",
    "x-real-ip": "xx.xx.xx.xx",
    "x-forwarded-for": "xx.xx.xx.xx",
    "x-forwarded-host": "ip-xx-xx-xx-xx.sbg.lb.ovh.net",
    "x-forwarded-port": "80",
    "x-forwarded-proto": "http",
    "x-scheme": "http",
    "cache-control": "max-age=0",
    "upgrade-insecure-requests": "1",
    "user-agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.190 Safari/537.36",
    "accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
    "accept-encoding": "gzip, deflate",
    "accept-language": "en-US,en;q=0.9"
  },
  "method": "GET",
  "body": "",
  "fresh": false,
  "hostname": "ip-xx-xx-xx-xx.sbg.lb.ovh.net",
  "ip": "xx.xx.xx.xx",
  "ips": [
    "xx.xx.xx.xx"
  ],
  "protocol": "http",
  "query": {},
  "subdomains": [
    "lb",
    "sbg",
    "ip-xx-xx-xx-xx"
  ],
  "xhr": false,
  "os": {
    "hostname": "echo-app-786947f46-qrlhk"
  },
  "connection": {}
}

```