# RKE with ArgoCD and MongoDB
A project that implements a RKE cluster with ArgoCD and MongoDB. The projects has a simple app (with pvc) to check that it is working correctly.

## Getting started

### Prerequisites

### installation

This project is implemented on 3 machines on Vsphere.

- Update your Linux System with the following commands:
    ```
    sudo apt-get -y update
    sudo apt-get -y install openssh-server
    sudo apt install -y sssd-ad sssd-tools realmd adcli nano vim tcpdump net-tools ca-certificates curl gnupg
    ```
- Disable swap and Modify sysctl entries with the following command:
    ```
    sudo vim /etc/fstab
    # Add comment to swap line
    sudo swapoff -a
    ```
- Set Hostname on Nodes:
    On evry node run the following commands to set and configure the nodes hostnames:
    ```
    sudo hostnamectl set-hostname <node-hostname>
    OR
    sudo nano /etc/hosts
    <node-ip> <node-hostname>
    ```
- Create rke user on master and workers servers with the following commands:
    ```
    adduser rke
    passwd rke
    sudo usermod -s /bin/bash rke
    ```
- Add rke user to sudoers:
    ```
    sudo nano /etc/sudoers.d/rke
    #add the folling line to the file:
    <!-- rke  ALL=(ALL:ALL) NOPASSWD: ALL -->
    ```
- Switch to rke user
  ```
  sudo su rke
  ```
- Create SSH key on the master server: 
  ```
  ssh-keygen -t rsa
  ```
- Copy the newly created ssh public key from master node to workers nodes manually or with the following function:
    ```
    for i in <node-1> <node-2> <node-x>  ; do
        ssh-copy-id rke@$i
    done
        
  # If you get Permission denied (publickey) while running ssh-copy-id ubuntu@remotemachine then on remote node edit the /etc/ssh/sshd_config file and update PasswordAuthentication from no to yes then restart the service using sudo systemctl restart sshd command
  ```
- Confirm you can login from your workstation (with user rke) - including login from master to itself --> ssh rke@<node_ip>
#### Docker
On all nodes
- Install Supported version of Docker
  (version 19.03) 
    ```
    curl https://releases.rancher.com/install-docker/19.03.sh | sudo sh
    sudo apt-get remove docker docker-engine docker.io containerd runc
    sudo apt-get update
    sudo apt-get install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update
    sudo apt-get install docker-ce=5:20.10.14~3-0~ubuntu-$(lsb_release -cs) docker-ce-cli=5:20.10.14~3-0~ubuntu-$(lsb_release -cs) containerd.io
    sudo docker run hello-world
    ```
- Start docker with the following command:
    ```
    sudo systemctl start docker
    sudo systemctl enable docker
    ```
    
- Add permission to rke user for executing docker command docker with the following command:
    ```
    sudo usermod -aG docker rke
    sudo systemctl restart docker
    ```    

- Configure sshd_config file to allow SSH TCP forwarding
    ```
    $ sudo nano /etc/ssh/sshd_config
    AllowTcpForwarding yes
    sudo systemctl restart sshd
    ```
#### RKE
On master
- Download RKE
  ```
  wget https://github.com/rancher/rke/releases/download/v1.4.17/rke_linux-amd64
  ```
- Upload the rke_linux-amd64 flder to /home/rke on the master server and run the following command:
    ```
    sudo chmod +x rke_linux-amd64
    sudo mv rke_linux-amd64 /usr/local/bin/rke
    rke --version
    ```
- Update cluster.yaml file
- Create the cluster with the following command and wait until the cluster creation done:
    ```
    rke up // by default the installation will look for manifest file named 'cluster.yml'
    ```
- Set KUBECONFIG variable to the file generated with the following command:
    ```
    export KUBECONFIG=./kube_config_cluster.yml
    OR choose your favourite way to communicate with kubernetes api (bashrc, kubectl config set-context etc..)
    ```
- Install kubectl if needed
  ```
  curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  sudo mv /home/rke/kubectl /usr/bin/kubectl
  sudo chmod +x /usr/bin/kubectl
  ```
- Check if the cluster setup was successful
  ```
  kubectl get nodes -o wide
  kubectl get pods -A
  ```
  the output should be your cluster nodes and all the cluster pods





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


## usage



        