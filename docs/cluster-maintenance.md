## Cluster Maintenance 11%

kubeadm allows us to upgrade our cluster components in the proper order, making sure to include important feature upgrades we might want to take advantage of in the latest stable version of Kubernertes. In this lesson, we will go through upgrading our cluster from version 1.13.5 to 1.14.1.

Get the version of the API server:

```bash
kubectl version --short
```
View the version of kubelet:

```bash
kubectl describe nodes 
```
View the version of controller-manager pod:

```bash
kubectl get po [controller_pod_name] -o yaml -n kube-system
```
Release the hold on versions of kubeadm and kubelet:

```bash
sudo apt-mark unhold kubeadm kubelet
```
Install version 1.14.1 of kubeadm:

```bash
sudo apt install -y kubeadm=1.14.1-00
```
Hold the version of kubeadm at 1.14.1:

```bash
sudo apt-mark hold kubeadm
Verify the version of kubeadm:
```
```bash
kubeadm version
```
Plan the upgrade of all the controller components:

```bash
sudo kubeadm upgrade plan
```
Upgrade the controller components:

```bash
sudo kubeadm upgrade apply v1.14.1
```
Release the hold on the version of kubectl:

```bash
sudo apt-mark unhold kubectl
```
Upgrade kubectl:

```bash
sudo apt install -y kubectl=1.14.1-00
```
Hold the version of kubectl at 1.14.1:

```bash
sudo apt-mark hold kubectl
```
Upgrade the version of kubelet:

```bash
sudo apt install -y kubelet=1.14.1-00
```
Hold the version of kubelet at 1.14.1:

```bash
sudo apt-mark hold kubelet
```

### OS Upgrade 


When we need to take a node down for maintenance, Kubernetes makes it easy to evict the pods on that node, take it down, and then continue scheduling pods after the maintenance is complete. Furthermore, if the node needs to be decommissioned, you can just as easily remove the node and replace it with a new one, joining it to the cluster.

 See which pods are running on which nodes:

```bash
kubectl get pods -o wide
```
Evict the pods on a node:

```bash
kubectl drain [node_name] --ignore-daemonsets
```
Watch as the node changes status:

```bash
kubectl get nodes -w
```
Schedule pods to the node after maintenance is complete:

```bash
kubectl uncordon [node_name]
```
Remove a node from the cluster:

```bash
kubectl delete node [node_name]
```
Generate a new token:

```bash
sudo kubeadm token generate
```
List the tokens:

```bash
sudo kubeadm token list
```
Print the kubeadm join command to join a node to the cluster:

```bash
sudo kubeadm token create [token_name] --ttl 2h --print-join-command
```

### Backup ETCD 

Backing up your cluster can be a useful exercise, especially if you have a single etcd cluster, as all the cluster state is stored there. The etcdctl utility allows us to easily create a snapshot of our cluster state (etcd) and save this to an external location. In this lesson, we’ll go through creating the snapshot and talk about restoring in the event of failure.

Get the etcd binaries:

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
```
Unzip the compressed binaries:

```bash
tar xvf etcd-v3.3.12-linux-amd64.tar.gz
```
Move the files into /usr/local/bin:

```bash
sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin
```
Take a snapshot of the etcd datastore using etcdctl:

```bash
sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
```
View the help page for etcdctl:

```bash
ETCDCTL_API=3 etcdctl --help
```
Browse to the folder that contains the certificate files:

```bash
cd /etc/kubernetes/pki/etcd/
```
View that the snapshot was successful:

```bash
ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```
Zip up the contents of the etcd directory:

```bash
sudo tar -zcvf etcd.tar.gz etcd
```
Copy the etcd directory to another server:

```bash
scp etcd.tar.gz cloud_user@18.219.235.42:~/
```
https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md
