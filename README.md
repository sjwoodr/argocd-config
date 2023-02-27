# argocd-config

Configuration yaml for Argo CD deployments

## Installation

I am currently using Kustomize for the deployment of ArgoCD. 

### Kustomize Installation Method

To apply the kustomize:

```bash
cd kustomize/overlay
kubectl apply -k .
```
