## Networking 11%


Kubernetes keeps networking simple for effective communication between pods, even if they are located on a different node. In this lesson, we’ll talk about pod communication from within a node, including how to inspect the virtual interfaces, and then get into what happens when a pod wants to talk to another pod on a different node.

See which node our pod is on:

```bash
kubectl get pods -o wide
```
Log in to the node:

```bash
ssh [node_name]
```
View the node's virtual network interfaces:

```bash
ifconfig
```
View the containers in the pod:

```bash
docker ps
```
Get the process ID for the container:

```bash
docker inspect --format '{{ .State.Pid }}' [container_id]
```
Use nsenter to run a command in the process's network namespace:

```bash
nsenter -t [container_pid] -n ip addr
```
https://kubernetes.io/docs/concepts/cluster-administration/networking/


A Container Network Interface (CNI) is an easy way to ease communication between containers in a cluster. The CNI has many responsibilities, including IP management, encapsulating packets, and mappings in userspace. In this lesson, we will cover the details of the Flannel CNI we used in our Linux Academy cluster and talk about the ways in which we simplified communication in our cluster.

Apply the Flannel CNI plugin:

```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

Helpful Links
Flannel Documentation
Installing Other CNI Plugins
Installing Addons in Kubernetes

https://github.com/coreos/flannel/blob/master/Documentation/kubernetes.md
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network
https://kubernetes.io/docs/concepts/cluster-administration/addons/


####Service Networking


Services allow our pods to move around, get deleted, and replicate, all without having to manually keep track of their IP addresses in the cluster. This is accomplished by creating one gateway to distribute packets evenly across all pods. In this lesson, we will see the differences between a NodePort service and a ClusterIP service and see how the iptables rules take effect when traffic is coming in.

YAML for the nginx NodePort service:

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  selector:
    app: nginx
```
Get the services YAML output for all the services in your cluster:

```bash
kubectl get services -o yaml
```
Try and ping the clusterIP service IP address:

```bash
ping 10.96.0.1
```
View the list of services in your cluster:

```bash
kubectl get services
```
View the list of endpoints in your cluster that get created with a service:

```bash
kubectl get endpoints
```
Look at the iptables rules for your services:

```bash
sudo iptables-save | grep KUBE | grep nginx
```

https://kubernetes.io/docs/concepts/services-networking/service/

### Ingress Rules and Load Balancers 


When handling traffic from outside sources, there are two ways to direct that traffic to your pods: deploying a load balancer, and creating an ingress controller and an Ingress resource. In this lesson, we will talk about the benefits of each and how Kubernetes distributes traffic to the pods on a node to reduce latency and direct traffic to the appropriate services within your cluster.

View the list of services:

```bash
kubectl get services
```
The load balancer YAML spec:

```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```
Create a new deployment:

```bash
kubectl run kubeserve2 --image=chadmcrowell/kubeserve2
```
View the list of deployments:

```bash
kubectl get deployments
```
Scale the deployments to 2 replicas:

```bash
kubectl scale deployment/kubeserve2 --replicas=2
```
View which pods are on which nodes:

```bash
kubectl get pods -o wide
```
Create a load balancer from a deployment:

```bash
kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer
```
View the services in your cluster:

```bash
kubectl get services
```
Watch as an external port is created for a service:

```bash
kubectl get services -w
```
Look at the YAML for a service:

```bash
kubectl get services kubeserve2 -o yaml
```
Curl the external IP of the load balancer:

```bash
curl http://[external-ip]
```
View the annotation associated with a service:

```bash
kubectl describe services kubeserve
```
Set the annotation to route load balancer traffic local to the node:

```bash
kubectl annotate service kubeserve2 externalTrafficPolicy=Local
```
The YAML for an Ingress resource:

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: service-ingress
spec:
  rules:
  - host: kubeserve.example.com
    http:
      paths:
      - backend:
          serviceName: kubeserve2
          servicePort: 80
  - host: app.example.com
    http:
      paths:
      - backend:
          serviceName: nginx
          servicePort: 80
  - http:
      paths:
      - backend:
          serviceName: httpd
          servicePort: 80
```
Edit the ingress rules:

```bash
kubectl edit ingress
```
View the existing ingress rules:

```bash
kubectl describe ingress
```
Curl the hostname of your Ingress resource:

```bash
curl http://kubeserve2.example.com
```

Helpful Links
Create an External Load Balancer
Ingress

https://kubernetes.io/docs/concepts/services-networking/ingress/


### Cluster DNS 

CoreDNS is now the new default DNS plugin for Kubernetes. In this lesson, we’ll go over the hostnames for pods and services. We will also discover how you can customize DNS to include your own nameservers.

View the CoreDNS pods in the kube-system namespace:

```bash
kubectl get pods -n kube-system
```
View the CoreDNS deployment in your Kubernetes cluster:

```bash
kubectl get deployments -n kube-system
```
View the service that performs load balancing for the DNS server:

```bash
kubectl get services -n kube-system
```
Spec for the busybox pod:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
```
View the resolv.conf file that contains the nameserver and search in DNS:

```bash
kubectl exec -it busybox -- cat /etc/resolv.conf
```
Look up the DNS name for the native Kubernetes service:

```bash
kubectl exec -it busybox -- nslookup kubernetes
```
Look up the DNS names of your pods:

```bash
kubectl exec -ti busybox -- nslookup [pod-ip-address].default.pod.cluster.local
```
Look up a service in your Kubernetes cluster:

```bash
kubectl exec -it busybox -- nslookup kube-dns.kube-system.svc.cluster.local
```
Get the logs of your CoreDNS pods:

```bash
kubectl logs [coredns-pod-name]
```
YAML spec for a headless service:

```bash
apiVersion: v1
kind: Service
metadata:
  name: kube-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: kubserve2
```
YAML spec for a custom DNS pod:

```bash
apiVersion: v1
kind: Pod
metadata:
  namespace: default
  name: dns-example
spec:
  containers:
    - name: test
      image: nginx
  dnsPolicy: "None"
  dnsConfig:
    nameservers:
      - 8.8.8.8
    searches:
      - ns1.svc.cluster.local
      - my.dns.search.suffix
    options:
      - name: ndots
        value: "2"
      - name: edns0
```

Helpful Links
DNS for Services and Pods https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
Debugging DNS Resolution https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/
Customizing DNS https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/
CoreDNS GitHub https://github.com/coredns/deployment/tree/master/kubernetes
Kubernetes DNS-Based Service Discovery https://github.com/kubernetes/dns/blob/master/docs/specification.md
Deploying CoreDNS using kubeadm https://coredns.io/2018/01/29/deploying-kubernetes-with-coredns-using-kubeadm/
