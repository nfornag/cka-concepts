
## Installation, Configuration & Validation 12%

Get the Docker gpg key:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Add the Docker repository:

```bash
sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```
Get the Kubernetes gpg key:

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
Add the Kubernetes repository:

```bash
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```
Update your packages:

```bash
sudo apt-get update
```
Install Docker, kubelet, kubeadm, and kubectl:

```bash
sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00
```
Hold them at the current version:

```bash
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```
Add the iptables rule to sysctl.conf:

```bash
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```
Enable iptables immediately:

```bash
sudo sysctl -p
```
Initialize the cluster (run only on the master):

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
Set up local kubeconfig:

```bash
mkdir -p $HOME/.kube
```
```bash
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
Apply Flannel CNI network overlay:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
Join the worker nodes to the cluster:

```bash
kubeadm join [your unique string from the kubeadm init command]
```
Verify the worker nodes have joined the cluster successfully:

```bash
kubectl get nodes
```
Compare this result of the kubectl get nodes command:

```bash
NAME                            STATUS   ROLES    AGE   VERSION
chadcrowell1c.mylabserver.com   Ready    master   4m18s v1.13.5
chadcrowell2c.mylabserver.com   Ready    none     82s   v1.13.5
chadcrowell3c.mylabserver.com   Ready    none     69s   v1.13.5
```
