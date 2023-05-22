# argocd-config

Configuration yaml for Argo CD deployments

## Installation Steps

### Uninstall k3s to start over
```
/usr/local/bin/k3s-uninstall.sh
```

### Install k3s without traefik
```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -
sudo chgrp -R adm /etc/rancher/k3s
sudo chmod -R g+rw /etc/rancher/k3s
```

### Install istio with demo profile
```
istioctl install --set profile=demo
```

### Install argocd
```
kubectl create ns argocd
cd ~/src/argocd-config/kustomize/istio-overlay
kubectl apply -k .
```

### Wait for argocd to come up before you proceed
```
kubectl -n argocd get pods
```

### Label namespaces for sidecar injection
```
kubectl create ns argocd
kubectl create ns development
kubectl create ns staging
kubectl label namespace default istio-injection=enabled
kubectl label namespace development istio-injection=enabled
kubectl label namespace staging istio-injection=enabled
```

### Protect argocd from outside world
```
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
EDIT authpolicy.yaml ---- change HOME_IP
kubectl apply -f authpolicy.yaml
git checkout authpolicy.yaml   # so you don't check in the IP
```

### Install cert-manager
```
kubectl create ns cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
kubectl get pods --namespace cert-manager
```

### Setup CloudFlare secret
```
export CLOUDFLARE_API_TOKEN=xxxxxxxxx
echo "
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
  namespace: istio-system
type: Opaque
stringData:
  api-token: $CLOUDFLARE_API_TOKEN
" | kubectl apply -f -
```

### Setup cert-manager Issuer
```
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
EDIT certmanager.yaml and change the email field
kubectl apply -f certmanager.yaml
git checkout certmanager.yaml  # so you don't check in your real email
```

### Create certs
```
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
kubectl apply -f certs.yaml
```

### Wait for certs to be issued / ready (~60 seconds)
```
kubectl get certs -A
kubectl describe -n istio-system certs argocd-n9oh-com
```

### Get initial argocd admin password
```
kubectl -n argocd get secret argocd-initial-admin-secret -o yaml | grep password: | awk -F: '{print $2}' | sed 's/ //g' | base64 -d
```

### Visit the argocd admin page
```
https://argocd.n9oh.com
SYNC all the things
```
