# rancher-microK8s

1. Prepare 4 VMs with Ubuntu <br>
   1 Controller Node <br>
   3 Worker Nodes <br>
   The VMs were provisioned in Azure Cloud <br>
   OS: Ubuntu 20.04 TLS <br>
   RAM: 4GB <br>
   CPU: 2 <br>
   
2. Access the VMs <br>
   A SSH Private Key is created to access the 4 VMs <br>
   All you need to do is execute in your terminal: <br>
   ssh -i {.pem file} {user}@{host} <br>
   ssh -i tml/k8s-dev.pem azureuser@k8s-node3 <br>
   
4. Install MicroK8s <br>
   $ sudo apt update <br>
   $ apt list --upgradable<br>
   $ sudo apt upgrade -y <br>
   $ sudo snap install microk8s --classic <br>
   microk8s (1.28/stable) v1.28.3 from Canonical✓ installed <br>
   https://microk8s.io/docs/services-and-ports to get the necessary ports to be open between the hosts <br>

   - Services binding to the default Host interface <br>
   $ sudo ufw allow 16443/tcp - API server <br>
   $ sudo ufw allow 10250/tcp - kubelet <br>
   $ sudo ufw allow 16255/tcp - kubelet <br>
   $ sudo ufw allow 25000/tcp - cluster-agent <br>
   $ sudo ufw allow 12379/tcp - etcd <br>
   $ sudo ufw allow 10257/tcp - kube-controller <br>
   $ sudo ufw allow 10259/tcp - kube-scheduler <br>
   $ sudo ufw allow 19001/tcp - dqlite <br>
   $ sudo ufw allow 4789/udp  - calico <br>
   
   - Services binding to the localhost interface <br>
   $ sudo ufw allow 10248/tcp - kubelet <br>
   $ sudo ufw allow 10249/tcp - kube-proxy <br>
   $ sudo ufw allow 10251/tcp - kube-scheduler <br>
   $ sudo ufw allow 10252/tcp - kube-controller <br>
   $ sudo ufw allow 10256/tcp - kube-proxy <br>
   $ sudo ufw allow 2380/tcp - etcd <br>
   $ sudo ufw allow 1338/tcp - containerd <br>
   
   ### To check the firewall rules applied: <br>
   $ sudo ufw enable <br>
   $ sudo ufw status <br>
   or <br>
   $ sudo ufw disable <br>
   
6. Check MicroK8s status <br>
   ```
   $ sudo su <br>
   # microk8s status
   ```
   
   
8. Install Rancher <br>
   
9. Deploy Keycloak 23.0.4 with PosgreSQL <br>
