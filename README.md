# rancher-microK8s

## 1. Prepare 4 VMs with Ubuntu <br>
   1 Controller Node <br>
   3 Worker Nodes <br>
   The VMs were provisioned in Azure Cloud <br>
   OS: Ubuntu 20.04 TLS <br>
   RAM: 4GB <br>
   CPU: 2 <br>

   10.0.0.4 k8s-controller <br>
   10.0.0.5 k8s-node1 <br>
   10.0.0.6 k8s-node2 <br>
   10.0.0.7 k8s-node3 <br>

   172.174.255.96 k8s-controller <br>
   74.235.148.231 k8s-node1 <br>
   20.127.155.255 k8s-node2 <br>
   172.203.243.162 k8s-node3 <br>
   
## 2. Access the VMs <br>
   A SSH Private Key is created to access the 4 VMs <br>
   All you need to do is execute in your terminal: <br>
   ```
   ssh -i {.pem file} {user}@{host}
   ```
## 3. Install MicroK8s <br>
 ```
   $ sudo apt update
   $ apt list --upgradable
   $ sudo apt upgrade -y
   $ sudo snap install microk8s --classic
 ```
   microk8s (1.28/stable) v1.28.3 from Canonical✓ installed <br>
   https://microk8s.io/docs/services-and-ports to get the necessary ports to be open between the hosts <br>

   - Services binding to the default Host interface <br>
   ```
   $ sudo ufw allow 16443/tcp - API server
   $ sudo ufw allow 10250/tcp - kubelet
   $ sudo ufw allow 16255/tcp - kubelet
   $ sudo ufw allow 25000/tcp - cluster-agent
   $ sudo ufw allow 12379/tcp - etcd
   $ sudo ufw allow 10257/tcp - kube-controller
   $ sudo ufw allow 10259/tcp - kube-scheduler
   $ sudo ufw allow 19001/tcp - dqlite
   $ sudo ufw allow 4789/udp  - calico
   ```
   - Services binding to the localhost interface <br>
   ```
   $ sudo ufw allow 10248/tcp - kubelet
   $ sudo ufw allow 10249/tcp - kube-proxy
   $ sudo ufw allow 10251/tcp - kube-scheduler
   $ sudo ufw allow 10252/tcp - kube-controller
   $ sudo ufw allow 10256/tcp - kube-proxy
   $ sudo ufw allow 2380/tcp - etcd
   $ sudo ufw allow 1338/tcp - containerd
   ```
   #### To check the firewall rules applied: <br>
   ```
   $ sudo ufw enable
   $ sudo ufw status
   ```
   #### To disable it
   ```
   $ sudo ufw disable
   ```
## 4. Check MicroK8s status <br>
   ```
   $ sudo su
   # microk8s status
   ```
You should see the following status:
```
root@k8s-controller:~# microk8s status
microk8s is running
high-availability: no
  datastore master nodes: 127.0.0.1:19001
  datastore standby nodes: none
```
## 5. Check Nodes
```
# microk8s kubectl get nodes
```
## 6. Attach nodes to cluster
```
# microk8s add node
```
To renew the token to join
```
# microk8s add node
```

## 5. Install Rancher <br>
   
## 6. Deploy Keycloak 23.0.4 with PosgreSQL <br>
