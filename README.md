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
   sudo snap install microk8s --classic --channel=1.27/stable
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
### 2. Add permission to Rancher to control the Kubernetes cluster
Run this command in all nodes on the cluster
```
sudo sh -c 'echo "--allow-privileged=true" /var/snap/microk8s/current/args/kube-apiserver'
```
Run this command on the controller node
```
microk8s stop daemon-apiserver && microk8s start daemon-apiserver
```
### 3. Install cert-manager v1.13.3 user by Rancher 
Run this command in all nodes on the cluster
```
microk8s helm3 repo add jetstack https://charts.jetstack.io
microk8s kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.3/cert-manager.crds.yaml
microk8s kubectl create namespace cert-manager
microk8s kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
microk8s helm3 install cert-manager --namespace cert-manager --version v1.13.3 jetstack/cert-manager
```

### 4. Install stable Rancher
```
microk8s kubectl create namespace cattle-system
microk8s kubectl label namespace cattle-system cattle-system.k8s.io/disable-validation=true
microk8s helm3 repo add rancher-latest https://releases.rancher.com/server-charts/latest
microk8s.helm3 repo update
```

### 5. Set /etc/hosts
Add the following line in your local hosts file
```
{Public.Ip.Address} rancher.tml.dev
```

### 6. Install command
Execute:
```
microk8s helm3 install rancher rancher-latest/rancher --namespace cattle-system  --set replicas=3 --set hostname=rancher.tml.dev
```
if you have some issue like this:
```
Error: INSTALLATION FAILED: chart requires kubeVersion: < 1.28.0-0 which is incompatible with Kubernetes v1.28.3
```
proceed with:
```
sudo snap refresh microk8s --channel=1.27/stable
```
and execute again the first command. You should have after this:
```
root@k8s-controller:~# microk8s helm3 install rancher rancher-latest/rancher --namespace cattle-system  --set replicas=3 --set hostname=rancher.tml.dev
NAME: rancher
LAST DEPLOYED: Mon Jan 15 02:41:17 2024
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rancher.tml.dev to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:

# echo https://rancher.tml.dev/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')

To get just the bootstrap password on its own, run:

# kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'

Happy Containering!
root@k8s-controller:~#
```


## III. Setup Keycloak 23.0.4
