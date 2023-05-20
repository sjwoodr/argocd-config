# argocd-config

Configuration yaml for Argo CD deployments

## Installation

I am currently using Kustomize for the deployment of ArgoCD. 

## Installation Steps

### Uninstall k3s to start over
/usr/local/bin/k3s-uninstall.sh

### Install k3s without traefik
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable=traefik" sh -
sudo chgrp -R steve /etc/rancher/k3s
sudo chmod g+rw /etc/rancher/k3s/k3s.yaml

### Install istio with demo profile
istioctl install --set profile=demo

### Label namespaces for sidecar injection
kubectl create ns argocd
kubectl label namespace default istio-injection=enabled
kubectl label namespace argocd istio-injection=enabled

### Install argocd
cd ~/src/argocd-config/kustomize/istio-overlay
kubectl apply -k .

### Protect argocd from outside world
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
EDIT authpolicy.yaml ---- change HOME_IP
kubectl apply -f authpolicy.yaml
git checkout authpolicy.yaml   # so you don't check in the IP

### Install cert-manager
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.0/cert-manager.yaml
kubectl get pods --namespace cert-manager

### Setup CloudFlare secret
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

### Setup cert-manager Issuer
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
EDIT certmanager.yaml and change the email field
kubectl apply -f certmanager.yaml
git checkout certmanager.yaml  # so you don't check in your real email

### Create certs
cd ~/src/argocd-config/kustomize/istio-overlay/argocd
kubectl apply -f certs.yaml

### Get initial argocd admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o yaml | grep password: | awk -F: '{print $2}' | sed 's/ //g' | base64 -d

