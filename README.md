# rancher-microK8s

1. Prepare 4 VMs with Ubuntu \n
   1 Controller Node \n
   3 Worker Nodes \n
   The VMs were provisioned in Azure Cloud \n
   OS: Ubuntu 20.04 TLS \n
   RAM: 4GB \n
   CPU: 2 \n
2. Access the VMs
   A SSH Private Key is created to access the 4 VMs
   All you need to do is execute in your terminal:
   ssh -i {.pem file} {user}@{host}
   ssh -i tml/k8s-dev.pem azureuser@k8s-node3
4. Install MicroK8s
   $ sudo apt update
   $ apt list --upgradable
   $ sudo apt upgrade -y
   $ sudo snap install microk8s --classic
   microk8s (1.28/stable) v1.28.3 from Canonical✓ installed
   https://microk8s.io/docs/services-and-ports to get the necessary ports to be open between the hosts
   Services binding to the default Host interface
   $ sudo ufw allow 16443/tcp - API server
   $ sudo ufw allow 10250/tcp - kubelet
   $ sudo ufw allow 16255/tcp - kubelet
   $ sudo ufw allow 25000/tcp - cluster-agent
   $ sudo ufw allow 12379/tcp - etcd
   $ sudo ufw allow 10257/tcp - kube-controller
   $ sudo ufw allow 10259/tcp - kube-scheduler
   $ sudo ufw allow 19001/tcp - dqlite
   $ sudo ufw allow 4789/udp  - calico
   Services binding to the localhost interface
   $ sudo ufw allow 10248/tcp - kubelet
   $ sudo ufw allow 10249/tcp - kube-proxy
   $ sudo ufw allow 10251/tcp - kube-scheduler
   $ sudo ufw allow 10252/tcp - kube-controller
   $ sudo ufw allow 10256/tcp - kube-proxy
   $ sudo ufw allow 2380/tcp - etcd
   $ sudo ufw allow 1338/tcp - containerd
   To check the firewall rules applied:
   $ sudo ufw enable
   $ sudo ufw status
   or
   $ sudo ufw disable
6. Check MicroK8s status
   $ sudo su
   ´´´ # microk8s status ´´´
7. Install Rancher
   
8. Deploy Keycloak 23.0.4 with PosgreSQL
