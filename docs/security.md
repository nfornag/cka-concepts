


## Security 12%

### Kubernetes Security Primitives 

Expanding on our discussion about securing the Kubernetes cluster, we’ll take a look at service accounts and user authentication. Also in this lesson, we will create a workstation for you to administer your cluster without logging in to the Kubernetes master server.

List the service accounts in your cluster:

```bash
kubectl get serviceaccounts
```
Create a new jenkins service account:

```bash
kubectl create serviceaccount jenkins
```
Use the abbreviated version of serviceAccount:

```bash
kubectl get sa
```
View the YAML for our service account:

```bash
kubectl get serviceaccounts jenkins -o yaml
```
View the secrets in your cluster:

```bash
kubectl get secret [secret_name]
```
The YAML for a busybox pod using the jenkins service account:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  serviceAccountName: jenkins
  containers:
  - image: busybox:1.28.4
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
    name: busybox
  restartPolicy: Always
```
Create a new pod with the service account:

```bash
kubectl apply -f busybox.yaml
```
View the cluster config that kubectl uses:

```bash
kubectl config view
```
View the config file:

```bash
cat ~/.kube/config
```
Set new credentials for your cluster:

```bash
kubectl config set-credentials chad --username=chad --password=password
```
Create a role binding for anonymous users (not recommended):

```bash
kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous
```
SCP the certificate authority to your workstation or server:

```bash
scp ca.crt cloud_user@[pub-ip-of-remote-server]:~/
```
Set the cluster address and authentication:

```bash
kubectl config set-cluster kubernetes --server=https://172.31.41.61:6443 --certificate-authority=ca.crt --embed-certs=true
```
Set the credentials for Chad:

```bash
kubectl config set-credentials chad --username=chad --password=password
```
Set the context for the cluster:

```bash
kubectl config set-context kubernetes --cluster=kubernetes --user=chad --namespace=default
```
Use the context:

```bash
kubectl config use-context kubernetes
```
Run the same commands with kubectl:

```bash
kubectl get nodes
```
Helpful Links
Securing the Cluster https://kubernetes.io/docs/concepts/cluster-administration/cluster-administration-overview/#securing-a-cluster
Authentication https://kubernetes.io/docs/reference/access-authn-authz/authentication/
Administer the Cluster via kubectl https://kubernetes.io/docs/reference/kubectl/overview/


### Cluster Authentication and Authorization 

Once the API server has determined who you are (whether a pod or a user), the authorization is handled by RBAC. In this lesson, we will talk about roles, cluster roles, role bindings, and cluster role bindings.

Create a new namespace:

```bash
kubectl create ns web
```
The YAML for a service role:

```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: web
  name: service-reader
rules:
- apiGroups: [""]
  verbs: ["get", "list"]
  resources: ["services"]
```
Create a new role from that YAML file:

```bash
kubectl apply -f role.yaml
```
Create a RoleBinding:

```bash
kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web
```
Run a proxy for inter-cluster communications:

```bash
kubectl proxy
```
Try to access the services in the web namespace:

```bash
curl localhost:8001/api/v1/namespaces/web/services
```
Create a ClusterRole to access PersistentVolumes:

```bash
kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes
```
Create a ClusterRoleBinding for the cluster role:

```bash
kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=web:default
```
The YAML for a pod that includes a curl and proxy container:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: curlpod
  namespace: web
spec:
  containers:
  - image: tutum/curl
    command: ["sleep", "9999999"]
    name: main
  - image: linuxacademycontent/kubectl-proxy
    name: proxy
  restartPolicy: Always
```
Create the pod that will allow you to curl directly from the container:

```bash
kubectl apply -f curl-pod.yaml
```
Get the pods in the web namespace:

```bash
kubectl get pods -n web
```
Open a shell to the container:

```bash
kubectl exec -it curlpod -n web -- sh
```
Access PersistentVolumes (cluster-level) from the pod:

```bash
curl localhost:8001/api/v1/persistentvolumes
```
Helpful Links:
Authorizationhttps://kubernetes.io/docs/reference/access-authn-authz/authorization/
RBAC https://kubernetes.io/docs/reference/access-authn-authz/rbac/
RoleBinding and ClusterRoleBinding https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding

### Configuring Network Policies 

Network policies allow you to specify which pods can talk to other pods. This helps when securing communication between pods, allowing you to identify ingress and egress rules. You can apply a network policy to a pod by using pod or namespace selectors. You can even choose a CIDR block range to apply the network policy. In this lesson, we’ll go through each of these options for network policies.

Download the canal plugin:

```bash
wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```
Apply the canal plugin:

```bash
kubectl apply -f canal.yaml
```
The YAML for a deny-all NetworkPolicy:

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```
Run a deployment to test the NetworkPolicy:

```bash
kubectl run nginx --image=nginx --replicas=2
```
Create a service for the deployment:

```bash
kubectl expose deployment nginx --port=80
```
Attempt to access the service by using a busybox interactive pod:

```bash
kubectl run busybox --rm -it --image=busybox /bin/sh
wget --spider --timeout=1 nginx
```
The YAML for a pod selector NetworkPolicy:

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: web
    ports:
    - port: 5432
```
Label a pod to get the NetworkPolicy:

```bash
kubectl label pods [pod_name] app=db
```
The YAML for a namespace NetworkPolicy:

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ns-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          tenant: web
    ports:
    - port: 5432
```
The YAML for an IP block NetworkPolicy:

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ipblock-netpolicy
spec:
  podSelector:
    matchLabels:
      app: db
  ingress:
  - from:
    - ipBlock:
        cidr: 192.168.1.0/24
```
The YAML for an egress NetworkPolicy:

```bash
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: egress-netpol
spec:
  podSelector:
    matchLabels:
      app: web
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: db
    ports:
    - port: 5432
```
Helpful Links:
Network Policies https://kubernetes.io/docs/concepts/services-networking/network-policies/
Declare Network Policies https://kubernetes.io/docs/tasks/administer-cluster/declare-network-policy/
Default Network Policies https://kubernetes.io/docs/concepts/services-networking/network-policies/#default-policies




### Creating TLS Certificates 

A Certificate Authority (CA) is used to generate TLS certificates and authenticate to your API server. In this lesson, we’ll go through certificate requests and generating a new certificate.

Find the CA certificate on a pod in your cluster:

```bash
kubectl exec busybox -- ls /var/run/secrets/kubernetes.io/serviceaccount
```
Download the binaries for the cfssl tool:

```bash
wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
```
Make the binary files executable:

```bash
chmod +x cfssl_linux-amd64 cfssljson_linux-amd64
```
Move the files into your bin directory:

```bash
sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
```
Check to see if you have cfssl installed correctly:

```bash
cfssl version
```
Create a CSR file:

```bash
cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "my-svc.my-namespace.svc.cluster.local",
    "my-pod.my-namespace.pod.cluster.local",
    "172.168.0.24",
    "10.0.34.2"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF
```
Create a CertificateSigningRequest API object:

```bash
cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: pod-csr.web
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF
```
View the CSRs in the cluster:

```bash
kubectl get csr
```
View additional details about the CSR:

```bash
kubectl describe csr pod-csr.web
```
Approve the CSR:

```bash
kubectl certificate approve pod-csr.web
```
View the certificate within your CSR:

```bash
kubectl get csr pod-csr.web -o yaml
```
Extract and decode your certificate to use in a file:

```bash
kubectl get csr pod-csr.web -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
```
Helpful Links
Managing TLS in the Cluster https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
Bootstrapping TLS for Your kubelets https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/
How kubeadm Manages Certificates https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/


### Secure Images 

Working with secure images is imperative in Kubernetes, as it ensures your applications are running efficiently and protecting you from vulnerabilities. In this lesson, we’ll go through how to set Kubernetes to use a private registry.

View where your Docker credentials are stored:

```bash
sudo vim /home/cloud_user/.docker/config.json
```
Log in to the Docker Hub:

```bash
sudo docker login
```
View the images currently on your server:

```bash
sudo docker images
```
Pull a new image to use with a Kubernetes pod:

```bash
sudo docker pull busybox:1.28.4
```
Log in to a private registry using the docker login command:

```bash
sudo docker login -u podofminerva -p 'otj701c9OucKZOCx5qrRblofcNRf3W+e' podofminerva.azurecr.io
```
View your stored credentials:

```bash
sudo vim /home/cloud_user/.docker/config.json
```
Tag an image in order to push it to a private registry:

```bash
sudo docker tag busybox:1.28.4 podofminerva.azurecr.io/busybox:latest
```
Push the image to your private registry:

```bash
docker push podofminerva.azurecr.io/busybox:latest
```
Create a new docker-registry secret:

```bash
kubectl create secret docker-registry acr --docker-server=https://podofminerva.azurecr.io --docker-username=podofminerva --docker-password='otj701c9OucKZOCx5qrRblofcNRf3W+e' --docker-email=user@example.com
```
Modify the default service account to use your new docker-registry secret:

```bash
kubectl patch sa default -p '{"imagePullSecrets": [{"name": "acr"}]}'
```
The YAML for a pod using an image from a private repository:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: acr-pod
  labels:
    app: busybox
spec:
  containers:
    - name: busybox
      image: podofminerva.azurecr.io/busybox:latest
      command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
      imagePullPolicy: Always
```
Create the pod from the private image:

```bash
kubectl apply -f acr-pod.yaml
```
View the running pod:

```bash
kubectl get pods
```
Helpful Links

```bash
Helpful Links
Images https://kubernetes.io/docs/concepts/containers/images/
Pull Images from a Private Registry https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
Configure Service Accounts https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/
Add ImagePullSecrets https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account
11 Ways (Not) to Get Hacked https://kubernetes.io/blog/2018/07/18/11-ways-not-to-get-hacked/
```
### Defining Security Contexts 


Defining security contexts allows you to lock down your containers, so that only certain processes can do certain things. This ensures the stability of your containers and allows you to give control or take it away. In this lesson, we’ll go through how to set the security context at the container level and the pod level.

Run an alpine container with default security:

```bash
kubectl run pod-with-defaults --image alpine --restart Never -- /bin/sleep 999999
```
Check the ID on the container:

```bash
kubectl exec pod-with-defaults id
```
The YAML for a container that runs as a user:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: alpine-user-context
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 405
```
Create a pod that runs the container as user:

```bash
kubectl apply -f alpine-user-context.yaml
```
View the IDs of the new pod created with container user permission:

```bash
kubectl exec alpine-user-context id
```
The YAML for a pod that runs the container as non-root:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: alpine-nonroot
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsNonRoot: true
```
Create a pod that runs the container as non-root:

```bash
kubectl apply -f alpine-nonroot.yaml
```
View more information about the pod error:

```bash
kubectl describe pod alpine-nonroot
```
The YAML for a privileged container pod:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      privileged: true
```
Create the privileged container pod:

```bash
kubectl apply -f privileged-pod.yaml
```
View the devices on the default container:

```bash
kubectl exec -it pod-with-defaults ls /dev
```
View the devices on the privileged pod container:

```bash
kubectl exec -it privileged-pod ls /dev
```
Try to change the time on a default container pod:

```bash
kubectl exec -it pod-with-defaults -- date +%T -s "12:00:00"
```
The YAML for a container that will allow you to change the time:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: kernelchange-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      capabilities:
        add:
        - SYS_TIME
```
Create the pod that will allow you to change the container’s time:

```bash
kubectl run -f kernelchange-pod.yaml
```
Change the time on a container:

```bash
kubectl exec -it kernelchange-pod -- date +%T -s "12:00:00"
```
View the date on the container:

```bash
kubectl exec -it kernelchange-pod -- date
```
The YAML for a container that removes capabilities:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: remove-capabilities
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      capabilities:
        drop:
        - CHOWN
```
Create a pod that’s container has capabilities removed:

```bash
kubectl apply -f remove-capabilities.yaml
```
Try to change the ownership of a container with removed capability:

```bash
kubectl exec remove-capabilities chown guest /tmp
```
The YAML for a pod container that can’t write to the local filesystem:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: readonly-pod
spec:
  containers:
  - name: main
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      readOnlyRootFilesystem: true
    volumeMounts:
    - name: my-volume
      mountPath: /volume
      readOnly: false
  volumes:
  - name: my-volume
    emptyDir:
```
Create a pod that will not allow you to write to the local container filesystem:

```bash
kubectl apply -f readonly-pod.yaml
```
Try to write to the container filesystem:

```bash
kubectl exec -it readonly-pod touch /new-file
```
Create a file on the volume mounted to the container:

```bash
kubectl exec -it readonly-pod touch /volume/newfile
```
View the file on the volume that’s mounted:

```bash
kubectl exec -it readonly-pod -- ls -la /volume/newfile
```
The YAML for a pod that has different group permissions for different pods:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: group-context
spec:
  securityContext:
    fsGroup: 555
    supplementalGroups: [666, 777]
  containers:
  - name: first
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 1111
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  - name: second
    image: alpine
    command: ["/bin/sleep", "999999"]
    securityContext:
      runAsUser: 2222
    volumeMounts:
    - name: shared-volume
      mountPath: /volume
      readOnly: false
  volumes:
  - name: shared-volume
    emptyDir:
```
Create a pod with two containers and different group permissions:

```bash
kubectl apply -f group-context.yaml
```
Open a shell to the first container on that pod:

```bash
kubectl exec -it group-context -c first sh
```
Helpful Links
Security Contexts https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

### Securing Persistent Key Value Store 

Secrets are used to secure sensitive data you may access from your pod. The data never gets written to disk because it's stored in an in-memory filesystem (tmpfs). Because secrets can be created independently of pods, there is less risk of the secret being exposed during the pod lifecycle.

View the secrets in your cluster:

```bash
kubectl get secrets
```
View the default secret mounted to each pod:

```bash
kubectl describe pods pod-with-defaults
```
View the token, certificate, and namespace within the secret:

```bash
kubectl describe secret
```
Generate a key for your https server:

```bash
openssl genrsa -out https.key 2048
```
Generate a certificate for the https server:


```bash
openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj /CN=www.example.com
```
Create an empty file to create the secret:

```bash
touch file
```
Create a secret from your key, cert, and file:

```bash
kubectl create secret generic example-https --from-file=https.key --from-file=https.cert --from-file=file
```
View the YAML from your new secret:

```bash
kubectl get secrets example-https -o yaml
```
Create the configMap that will mount to your pod:

```bash
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
data:
  my-nginx-config.conf: |
    server {
        listen              80;
        listen              443 ssl;
        server_name         www.example.com;
        ssl_certificate     certs/https.cert;
        ssl_certificate_key certs/https.key;
        ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers         HIGH:!aNULL:!MD5;

        location / {
            root   /usr/share/nginx/html;
            index  index.html index.htm;
        }

    }
  sleep-interval: |
    25
```
The YAML for a pod using the new secret:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: example-https
spec:
  containers:
  - image: linuxacademycontent/fortune
    name: html-web
    env:
    - name: INTERVAL
      valueFrom:
        configMapKeyRef:
          name: config
          key: sleep-interval
    volumeMounts:
    - name: html
      mountPath: /var/htdocs
  - image: nginx:alpine
    name: web-server
    volumeMounts:
    - name: html
      mountPath: /usr/share/nginx/html
      readOnly: true
    - name: config
      mountPath: /etc/nginx/conf.d
      readOnly: true
    - name: certs
      mountPath: /etc/nginx/certs/
      readOnly: true
    ports:
    - containerPort: 80
    - containerPort: 443
  volumes:
  - name: html
    emptyDir: {}
  - name: config
    configMap:
      name: config
      items:
      - key: my-nginx-config.conf
        path: https.conf
  - name: certs
    secret:
      secretName: example-https
```
Describe the nginx conf via ConfigMap:

```bash
kubectl describe configmap
```
View the cert mounted on the container:

```bash
kubectl exec example-https -c web-server -- mount | grep certs
```
Use port forwarding on the pod to server traffic from 443:

```bash
kubectl port-forward example-https 8443:443 &
```
Curl the web server to get a response:

```bash
curl https://localhost:8443 -k
```
Helpful Links
Secrets https://kubernetes.io/docs/concepts/configuration/secret/
