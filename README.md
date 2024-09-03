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

## Access ArgoCD UI

Get the default admin password for ArgoCD from the secret:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Get the allocated IP address for the ingress that you defined:
```
alex@lab:~$ kubectl get ingress -n argocd
NAME                 CLASS     HOSTS           ADDRESS         PORTS   AGE
argocd-server        traefik   argo.asdfx.us   192.168.0.200   80      5d16h
```

Setup a DNS entry (or an entry in `/etc/hosts`) to configure your argo domain to point to the IP address for the ingress.

## Uninstallation of K3s
You can completly uninstall k3s quickly and easily with:
```
sudo /usr/local/bin/k3s-uninstall.sh
```
