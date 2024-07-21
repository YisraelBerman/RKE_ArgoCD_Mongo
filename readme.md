# RKE with ArgoCD and MongoDB
A project that implements a RKE cluster with ArgoCD and MongoDB. The projects has a simple app (with pvc) to check that it is working correctly.

## Getting started

### Prerequisites

### installation

## usage




rke built in ingress-nginx ( ingress image: rancher/nginx-ingress-controller:nginx-1.9.4-rancher1) 
with editing /etc/hosts




kubectl apply -f pvc.yaml -n apps
kubectl apply -f populate-pvc-job.yaml -n apps
kubectl apply -f deployment.yaml -n apps
kubectl apply -f service.yaml -n apps











helm repo add nfs-ganesha-server-and-external-provisioner https://kubernetes-sigs.github.io/nfs-ganesha-server-and-external-provisioner/
helm repo update
helm install my-nfs nfs-ganesha-server-and-external-provisioner/nfs-server-provisioner

sudo apt-get install nfs-common

on all nodes:
sudo apt-get update
sudo apt-get install nfs-common rpcbind
sudo rpc.statd
sudo update-rc.d nfs-common defaults


#metalLB:
#kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml
#kubectl apply -f lb-config.yaml



export KUBECONFIG=~/kube_config_cluster.yml
alias k=kubectl
alias kucns='kubectl config set-context --current --namespace '









argocd:
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd --namespace argocd
need to change values:

server:
  # Disable HTTPS if needed
  extraArgs:
    - --insecure
  ingress:
    enabled: true
    annotations:
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
    
    paths:
      - /
    tls: []


to get to argocd:
  <host>/argo-cd
  

kubectl -n apps get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d





        