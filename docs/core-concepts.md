

```bash
kubectl auth can-i list secrets --namespace dev --as dave
```

```bash
kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db -n finance --dry-run -o yaml
```

```bash
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
```

```bash
kubectl create deploy --image=kodekloud/webapp-color webapp  --dry-run -o yaml
kubectl create deployment ngnix --image=nginx --dry-run -o yaml
```

```bash
kubectl set image deployment/clint-deployment clint=stephengride/multiclient:v5
kubectl scale deployment/webapp --replicas=3
kubectl scale rs new-replica-set --replicas=5
kubectl run --generator=run-pod/v1
kubectl run --generator=run/v1
kubectl run --generator=deployment/v1beta1
kubectl run --generator=deployment/apps.v1beta1
kubectl run --generator=job/v1
kubectl run --generator=cronjob/v1beta1
kubectl run --generator=cronjob/v2alpha1
```

## Secrets

```bash
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com
```
## ConfigMaps


#####static pods##

```bash
ps -aux|grep kubelet |grep yaml
```
```bash
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
```

### Rollouts##
```bash
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
kubectl rollout undo deployment/myapp-deployment
```

#####upgrade os##

```bash
kubectl drain node01 --ignore-daemonsets
kubectl uncordon node01
kubectl cordon node01
apt install kubeadm=1.12.0-00 and then apt install kubelet=1.12.0-00 and then kubeadm upgrade node config --kubelet-version $(kubelet --version | cut -d ' ' -f 2)
```
#####backup##
```bash
ETCDCTL_API=3 etcdctl \
etcdctl snapshot save /tmp/snapshot-pre-boot.db -n ingress-space
```


TLS
```bash
openssl x509 -in /etc/kubernetes/pki/etcd/server.crt -text -noout
openssl x509 -in  server.crt -text
```


#####Security Contexts##

```bash
kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
kubectl exec -it ubuntu-sleeper -- date -s '19 APR 2012 11:14:00'
```

#####Networking##

```bash
/sbin/ifconfig | grep HWaddr

ip link show ens3

netstat -nplt

 netstat -anp | grep etcd

ps -aux|grep kubelet |grep network


cd /etc/cni/net.d/

kubectl get pods -n kube-system

https://www.weave.works/docs/net/latest/kubernetes/kube-addon/


ip addr
kubectl logs weave-net-kkxr7 weave -n kube-system |grep ipalloc-range

ps -aux | grep kube-api |grep service-cluster-ip-range

https://medium.com/@lucky.cs025/kubertnetes-ingress-controller-setup-using-kop-nginx-ingress-6c3da8dde3f8

```
#####Ingress##

```bash
kubectl get ingress --all-namespaces


kubectl edit ingress --namespace app-space

pay-service


webapp-pay

8080/TCP

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282


kubectl create namespace ingress-space



kubectl create configmap nginx-configuration -n ingress-space

kubectl create serviceaccount ingress-serviceaccount -n ingress-space

```
DevOps Engineer

```bash
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /wear
        backend:
          serviceName: wear-service
          servicePort: 8080
      - path: /watch
        backend:
          serviceName: video-service
          servicePort: 8080
```
master $

```bash

sudo gpasswd -a vagrant docker

netstat -nltp
```
```bash
kubectl create secret docker-registry private-reg-cred --docker-username=dock_user --docker-password=dock_password --docker-server=myprivateregistry.com:5000 --docker-email=dock_user@myprivateregistry.com


```

```bash
ip link show ens3
 ls /etc/cni/net.d/
```
```bash

openssl genrsa -out nyarlagadda.key 2048
openssl req -new -key nyarlagadda.key -subj "/CN=nyarlagadda" -out nyarlagadda.csr
::::::::::::::
csr.yml
::::::::::::::
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: my-svc.my-namespace
spec:
  request: $(cat nyarlagadda | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

kubectl certificate approve nyarlagadda

kubectl get csr nyarlagadda -o yaml | awk '/certificate:/ {print $2}' |base64 --decode > nyarlagadda.crt
export CLUSTER_NAME=grep cluster: .kube/config | awk '/cluster:/ {print $2}' | grep -v cluster:
export SERVER=grep server: .kube/config | awk '/server:/ {print $2}' | grep -v server:
grep certificate-authority-data: .kube/config | awk '/certificate-authority-data:/ {print $2}' | grep -v certificate-authority-data: |base64 --decode > ca.crt

kubectl config --kubeconfig=config-demo set-credentials nyarlagadda --client-certificate=nyarlagadda.crt --client-key=nyarlagadda.key
kubectl config --kubeconfig=config-demo set-cluster $CLUSTER_NAME --server=$SERVER --certificate-authority=ca.crt
kubectl config --kubeconfig=config-demo set-context cka-practice@gke --cluster=$CLUSTER_NAME --namespace=kube-system --user=nyarlagadda

::::::::::::::
crb.yml
::::::::::::::
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: role-grantor-binding
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: nyarlagadda 
```
```bash

https://8gwifi.org/docs/kube-rbac.jsp
```

```bash
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: dev-role-binding
  namespace: dev
subjects:
- kind: User
  name: user1
  apiGroup: ""
roleRef:
  kind: Role
  name: devlopment
  apiGroup: ""
```
```bash

kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  namespace: stage
  name: staging
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```



## Kubectl advanced commands

```bash
kubectl get nodes -o json > /opt/outputs/nodes.json
kubectl get node node01 -o json > /opt/outputs/node01.json
kubectl get nodes -o=jsonpath='{.items[*].metadata.name}' > /opt/outputs/node_names.txt
kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.osImage}' > /opt/outputs/nodes_os_x43kj56.txt
kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.users[*].name}" > /opt/outputs/users.txt
kubectl get pv --sort-by=.spec.capacity.storage
kubectl get pv --sort-by=.spec.capacity.storage > /opt/outputs/storage-capacity-sorted.txt
kubectl get pv --sort-by=.spec.capacity.storage -o=custom-columns=NAME:.metadata.name,CAPACITY:.spec.capacity.storage > /opt/outputs/pv-and-capacity-sorted.txt
kubectl config view --kubeconfig=my-kube-config -o jsonpath="{.contexts[?(@.context.user=='aws-user')].name}" > /opt/outputs/aws-context-name
```

```bash
::::::::::::::
pv1.yml
::::::::::::::
apiVersion: v1
kind: PersistentVolume
metadata:
  name: redis01
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/redis01"
---
::::::::::::::
redis.yml
::::::::::::::
apiVersion: v1
kind: Service
metadata:
  name: redis-cluster-service
  labels:
    app: redis
spec:
  ports:
  - port: 6379
    name: client
    targetPort: 6379
  - port: 16379
    name: gossip
    targetPort: 16379
  clusterIP: None
  selector:
    app: redis
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: redis-cluster
spec:
  selector:
    matchLabels:
      app: redis # has to match .spec.template.metadata.labels
  serviceName: "redis-cluster-service"
  replicas: 6 # by default is 1
  template:
    metadata:
      labels:
        app: redis # has to match .spec.selector.matchLabels
    spec:
      containers:
      - name: redis
        image: redis:5.0.1-alpine
        command: ["sudo /conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        imagePullPolicy: Always
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: data
          mountPath: /data
          readOnly: false
      volumes:
      - name: conf
        configMap:
           name: redis-cluster-configmap
  volumeClaimTemplates:
  - metadata:
     name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi



master $ kubectl get configmap redis-cluster-configmap -o yaml
apiVersion: v1
data:
  redis.conf: |-
    cluster-enabled yes
    cluster-require-full-coverage no
    cluster-node-timeout 15000
    cluster-config-file /data/nodes.conf
    cluster-migration-barrier 1
    appendonly yes
    protected-mode no
  update-node.sh: |
    #!/bin/sh
    REDIS_NODES="/data/nodes.conf"
    sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" ${REDIS_NODES}
    exec "$@"
kind: ConfigMap
metadata:
  creationTimestamp: "2019-09-10T17:34:01Z"
  name: redis-cluster-configmap
  namespace: default
  resourceVersion: "5145"
  selfLink: /api/v1/namespaces/default/configmaps/redis-cluster-configmap
  uid: 2db8dfdb-d3f1-11e9-a0d2-0242ac11001b
```
