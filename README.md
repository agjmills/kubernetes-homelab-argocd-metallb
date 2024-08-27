# Bootstrap for homelab K3s cluster with ArgoCD

This repository assumes a fairly fresh x86_64 server, probably Ubuntu 22.04 LTS, with a pool of unallocated IP addresses at it's disposal.

In this example, let's assume 192.168.0.200-192.168.0.250 is unallocated and can be used to provide ingress to services we will advertise from the kubernetes deployment.

## Installation of Kubernetes (k3s)

Install K3s on the server:

```
curl -sfL https://get.k3s.io | sh -
```

Take a copy of the kubeconfig found in `etc/rancher/k3s/k3s.yaml` and modify the server field to reflect the remote IP address of the server you are configuring.

Store this somewhere, such as `~/.kube/lab.yaml` and then configure `kubectl` to use it with `export KUBECONFIG=~/.kube/lab.yaml`.

## Install Metallb, and ArgoCD

Install MetalLB:

```
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb --namespace metallb-system --create-namespace
kubectl apply -f metallb/config.yaml # to adjust the pool of IP addresses that metallb can allocate, modify the addresses attribute in config.yaml
```

Install ArgoCD:
```
helm repo add argo https://argoproj.github.io/argo-helm
helm install argocd argo/argo-cd --namespace argocd --create-namespace -f argo/values.yaml
```

Get the default admin password for ArgoCD:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Get the allocated IP address for the ArgoCD Load Balancer:
```
kubectl get service -n argocd
```

Look for a service called `argocd-server` with a Type of `LoadBalancer` and find the External-IP. You should be able to go to that IP address, and login with the username `admin` and the password you got from the penultimate step.

## Uninstallation of K3s
You can completly uninstall k3s quickly and easily with:
```
sudo /usr/local/bin/k3s-uninstall.sh
```
