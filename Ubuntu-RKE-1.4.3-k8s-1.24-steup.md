# Setup Kubernetes 1.24.10 cluster with RKE

>## RKE Kubernetes Installation Guide
> https://rke.docs.rancher.com/installation

## Servers preparations Chapter:
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
### (Eyal) why not using sudo usermod -aG sudo rke ???
    sudo nano /etc/sudoers.d/rke

    #add the folling line to the file:
    <!-- rke  ALL=(ALL:ALL) NOPASSWD: ALL -->


sudo su rke

- Create SSH key on the master server: 


### i think this needs to be done logged in as rke
### do on all nodes

    ssh-keygen -t rsa
   
- Copy the newly created ssh public key from master node to workers nodes manually or with the following function:

    ```
    for i in <node-1> <node-2> <node-x>  ; do
        ssh-copy-id rke@$i
    done
    ```    
### (Yisrael) If you get Permission denied (publickey) while running ssh-copy-id ubuntu@remotemachine then on remote node edit the /etc/ssh/sshd_config file and update PasswordAuthentication from no to yes then restart the service using sudo systemctl restart sshd command
### (Eyal) including master

- Confirm you can login from your workstation (with user rke) --> ssh rke@<node_ip>

<!-- - Enable required Kernel modules: (Not neccessary)
    ```
    for module in br_netfilter ip6_udp_tunnel ip_set ip_set_hash_ip ip_set_hash_net iptable_filter iptable_nat iptable_mangle iptable_raw nf_conntrack_netlink nf_conntrack nf_conntrack_ipv4   nf_defrag_ipv4 nf_nat nf_nat_ipv4 nf_nat_masquerade_ipv4 nfnetlink udp_tunnel veth vxlan x_tables xt_addrtype xt_conntrack xt_comment xt_mark xt_multiport xt_nat xt_recent xt_set  xt_statistic xt_tcpudp;
     do
       if ! lsmod | grep -q $module; tRhen
         echo "module $module is not present";
         sudo modprobe $module
       fi;
    done
    ``` -->
>## Docker Chapter - install on all nodes
- Install Supported version of Docker
    - version 19.03: 
        ```
        curl https://releases.rancher.com/install-docker/19.03.sh | sudo sh
### (Eyal): Package docker-ce is not available
    -  version 20.10:
        curl https://releases.rancher.com/install-docker/20.10.sh | sudo sh
### (Eyal): ERROR: '20.10.24' not found amongst apt-cache madison results

### (yisrael) instal docker 
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



> docker version above 20.10.x not supported! (version 23.0.X not supported)
### (Eyal) used this to uninstall older versions:  for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done

### (Eyal) check compatibility at: https://www.suse.com/suse-rancher/support-matrix/all-supported-versions/rancher-v2-8-2/
### (Eyal) curl https://releases.rancher.com/install-docker/<version-number>.sh | sh

### (Eyal) Install on all hosts???


### (Nevo) Install docker-compose 
### (Nevo) sudo curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)"  -o /usr/local/bin/docker-compose
### (Nevo) sudo mv /usr/local/bin/docker-compose /usr/bin/docker-compose
### (Nevo) sudo chmod +x /usr/bin/docker-compose
- Start docker with the following command:
    ```
    sudo
     systemctl start docker
    sudo systemctl enable docker
    
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
### (Eyal): should be ssh in ubuntu


## RKE setup Chapter:
### on master
- Download RKE [Release v1.4.3](https://github.com/rancher/rke/releases/download/v1.5/rke_linux-amd64) from RKE github page or insted use the rke_linux-amd64 folder that exists in this folder.
```
wget https://github.com/rancher/rke/releases/download/v1.4.17/rke_linux-amd64
```
```


### (Eyal) Preffer https://github.com/rancher/rke/releases/download/v1.4.17/rke_linux-amd64

- Upload the rke_linux-amd64 flder to /home/rke on the master server and run the following command:
    ```
    sudo chmod +x rke_linux-amd64
    sudo mv rke_linux-amd64 /usr/local/bin/rke
### (Eyal) should be sudo mv rke_linux-amd64 /usr/local/bin/rke
### check
    rke --version
- Upload the cluster.yml file that exists in this folder and change the nodes ip's addresses to your cluster nodes addresses. If you want to create new cluster yaml run the following command and insert your cluster requirements.
    ```
    rke config --name cluster.yml
    ```
    

- Create the cluster with the following command and wait until the cluster creation done:
    ```
    rke up // node: by default the installation will look for manifest file named 'cluster.yml'
    ```
- Set KUBECONFIG variable to the file generated with the following command:
    ```
    export KUBECONFIG=./kube_config_cluster.yml
    OR choose your favourite way to communicate with kubernetes api (bashrc, kubectl config set-context etc..)
    ```

### (Nevo) Install kubectl   
### (Nevo) curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
### (Nevo) sudo mv /home/rke/kubectl /usr/bin/kubectl
### (Nevo) sudo chmod +x /usr/bin/kubectl


- To check if the cluster setup was successful (ensure that you have 'kubectl' binary on your machine)- run the following commands:
    ```
    kubectl get nodes -o wide
    kubectl get pods -A
    ```
    the output should be your cluster nodes and all the cluster pods

>## Notes:
> [Install Kubernetes Dashboard](https://computingforgeeks.com/how-to-install-kubernetes-dashboard-with-nodeport/)

## Ingress-Fix:
kubectl create secret tls ingress-nginx-admission --namespace=ingress-nginx --cert=tsloc.crt --key=tsloc.key
get & replace the deamon set with hostNetwork: true in spec section

## Rancher deploy with Helm:
helm install rancher rancher-2.7.1/rancher \
  --namespace cattle-system \
  --set hostname=rancher24.ts.loc \
  --set bootstrapPassword=admin

helm install rancher rancher-stable/rancher --namespace cattle-system --set hostname=rancher24.ts.loc --set bootstrapPassword=admin


### (Nevo) helm install
### (Nevo) curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
### (Nevo) chmod 700 get_helm.sh
### (Nevo) ./get_helm.sh
### (Nevo) Deploying the Dashboard UI
### (Nevo) helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
### (Nevo) helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
### (Nevo)  kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443

## argo install
need to install with helm chart and change ingress value to true.
mkdir helm-charts
cd helm-charts
download helm chart
ccreate lab-values.yaml file with the overrides:

non-HA
```
ingress:
  enabled: true
  controller: generic
  ingressClassName: nginx
  hostname: # here put a worker of the node.
  path: /argo-cd
  pathType: Prefix
```
argo will create the certificate with this name:
 TLS certificate will be retrieved from a TLS secret `argocd-server-tls`

 so we need to use this line for generating the certificate secret:
kubectl create secret tls argocd-server-tls --namespace=ingress-nginx --cert=tsloc.crt --key=tsloc.key

after creating the file:
helm install argo-cd . -f lab-values.yaml 


another way is to use subdomains. for that we need to add a dns entry to point a subdomain to the ingress controller

```
ingress:
  enabled: true
  controller: generic
  ingressClassName: nginx
  hostname: argo-cd.uri.kuber.hcg.local # put here the dns subdomain of the ingress controller
  path: /argo-cd
  pathType: Prefix
```