# rancher-microK8s

## I. Setup Microk8s Cluster

### 1. Prepare 4 VMs with Ubuntu <br>
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

   172.208.98.42 k8s-controller <br>
   74.235.148.231 k8s-node1 <br>
   20.127.155.255 k8s-node2 <br>
   172.203.243.162 k8s-node3 <br>
   
### 2. Access the VMs <br>
   A SSH Private Key is created to access the 4 VMs <br>
   All you need to do is execute in your terminal: <br>
   ```
   ssh -i {.pem file} {user}@{host}
   ```
### 3. Install MicroK8s <br>
 ```
   sudo apt update
   apt list --upgradable
   sudo apt upgrade -y
   sudo snap install microk8s --classic
 ```
   microk8s (1.28/stable) v1.28.3 from Canonical✓ installed <br>
   https://microk8s.io/docs/services-and-ports to get the necessary ports to be open between the hosts <br>

   - Services binding to the default Host interface <br>
   ```
   sudo ufw allow 16443/tcp
   sudo ufw allow 10250/tcp
   sudo ufw allow 16255/tcp
   sudo ufw allow 25000/tcp
   sudo ufw allow 12379/tcp
   sudo ufw allow 10257/tcp
   sudo ufw allow 10259/tcp
   sudo ufw allow 19001/tcp
   sudo ufw allow 4789/udp
   ```
   - Services binding to the localhost interface <br>
   ```
   sudo ufw allow 10248/tcp
   sudo ufw allow 10249/tcp
   sudo ufw allow 10251/tcp
   sudo ufw allow 10252/tcp
   sudo ufw allow 10256/tcp
   sudo ufw allow 2380/tcp
   sudo ufw allow 1338/tcp
   ```
   #### To check the firewall rules applied: <br>
   ```
   sudo ufw enable
   sudo ufw status
   ```
   #### To disable it
   ```
   sudo ufw disable
   ```
### 4. Check MicroK8s status <br>
   ```
   sudo su
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
### 5. Check Nodes
It's important that all the nodes must discover themselves in the network. I advice to set in all nodes the /etc/hosts file like this:

```
   10.0.0.4 k8s-controller
   10.0.0.5 k8s-node1
   10.0.0.6 k8s-node2
   10.0.0.7 k8s-node3
```

### 6. Attach nodes to cluster
```
microk8s add-node
```
To renew the token to join
```
microk8s add node
```

After joining a new node every ~3min you should receive the following message: 

```
The node has joined the cluster and will appear in the nodes list in a few seconds.

This worker node gets automatically configured with the API server endpoints.
If the API servers are behind a loadbalancer please set the '--refresh-interval' to '0s' in:
    /var/snap/microk8s/current/args/apiserver-proxy
and replace the API server endpoints with the one provided by the loadbalancer in:
    /var/snap/microk8s/current/args/traefik/provider.yaml
```


To grant that you have all the nodes connected in the MicroK8s cluster:
```
root@k8s-controller:~# microk8s kubectl get nodes
NAME             STATUS   ROLES    AGE     VERSION
k8s-node2        Ready    <none>   8m39s   v1.28.3
k8s-node1        Ready    <none>   11m     v1.28.3
k8s-controller   Ready    <none>   16m     v1.28.3
k8s-node3        Ready    <none>   5m12s   v1.28.3
```

### 6. Make Cluster Fault Tolerant
Run this command on all hosts in the cluster
```
echo "failure-domain=42" > /var/snap/microk8s/current/args/ha-conf
```
Restart MicroK8s in Controller Node
```
microk8s stop && microk8s start
```
Restart MicroK8s in Worker Nodes
```
snap stop microk8s && snap start microk8s
```

## II. Setup Rancher

### 1. Enable Microk8s plugins for Rancher
```
microk8s enable rbac dns dashboard storage ingress helm3 metallb prometheus
```
To set MetalLB we can consider:
```
Enabling MetalLB
Enter each IP address range delimited by comma (e.g. '10.64.140.43-10.64.140.49,192.168.0.105-192.168.0.111'): 10.64.140.43-10.64.140.49
```

## III. Setup Keycloak 23.0.4
