


```bash
kubectl run --generator=run-pod/v1 nginx-pod --image=nginx:alpine
kubectl run --generator=run-pod/v1 redis --image=redis:alpine -l tier=db 
kubectl run --generator=run-pod/v1 temp-bus --image=redis:alpine -n finance
kubectl run --generator=run-pod/v1 nginx --image=nginx
kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml
```

```bash
kubectl run --generator=deployment/v1beta1 hr-web-app --image=kodekloud/webapp-color --replicas=2
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run -o yaml
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml
kubectl run --generator=deployment/v1beta1 nginx --image=nginx --dry-run --replicas=4 -o yaml > nginx-deployment.yaml
```

```bash
kubectl create deployment --image=kodekloud/webapp-color webapp 
kubectl create deployment --image=kodekloud/webapp-color webapp --replicas=4 --dry-run -o yaml
kubectl create deployment webapp --image=kodekloud/webapp-color. The scale the webapp to 3 using command kubectl scale deployment/webapp --replicas=3
```

```bash
kubectl create deployment --image=nginx nginx
kubectl set image deployment/clint-deployment clint=stephengride/multiclient:v5
kubectl scale deployment/webapp --replicas=3
```

```bash
kubectl expose pod redis --port=6379 --name redis-service
kubectl expose deployment hr-web-app --type=NodePort --port=8080 --name=hr-web-app-service --dry-run -o yaml > webapp-service.yaml
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

```bash
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
``` 
```bash
node01 $ ps -aux|grep kubelet |grep yaml
root     18253  2.1  2.4 887200 98236 ?        Ssl  14:11   0:10 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=cgroupfs --cni-bin-dir=/opt/cni/bin --cni-conf-dir=/etc/cni/net.d --network-plugin=cni
node01 $ vi /var/lib/kubelet/config.yaml
node01 $ cd /etc/just-to-mess-with-you/
node01 $ ls
greenbox.yaml
node01 $ rm greenbox.yaml
```

### Rollouts##
```bash
kubectl run --restart=Never --image=busybox:1.28.4 static-busybox --dry-run -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
kubectl rollout status deployment/myapp-deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
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


/etc/kubernetes/pki/users
/etc/kubernetes/pki/users/aws-user/
/etc/kubernetes/pki/users/dev-user/


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
```
```bash
kubectl config --kubeconfig=config-demo set-credentials nyarlagadda --client-certificate=/home/ynraju4/nyarlagadda.crt --client-key=/home/ynraju4/nyarlagadda.key
```
```bash

apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: nyarlagadda
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ2VUQ0NBV0VDQVFBd05ERVZNQk1HQTFVRUF3d01ibmxoY214aFoyRmtaR0VnTVJzd0dRWURWUVFLREJKRQpaWFpQY0hNdGJubGhjbXhoW
jJGa1pHRXdnZ0VpTUEwR0NTcUdTSWIzRFFFQkFRVUFBNElCRHdBd2dnRUtBb0lCCkFRREMyUXA4eXdseFBDRzAvVExKRTM5TXlhRFViL1FYa3pUNmJWYXNsRGEyMXcwZVhVY3VERFdRa2hUN0xNZlIKQWNMVkJtMUI5cWdhb
VoxeUZOR2QwdThaUUtqWkduTUlrRzdRVEs1dU55Q1J5UnpaYk15dFpyY255cWMwZ0lrZQo3OWUrM28ySXdoUFBtbnpjeGRRYUxURWVqMGp4NWNTWjBXSGFnR3B0dFpnUHlqeDJSMysrVkZNeUlMUDQvak04CmRrRzdZM1FLR
WI2N0JGcGpoSlcxS3dGeHo4ZzMvcjluaC9BcW9IWXR3OHpINDFEM3hodng2b0NJVnZrYWRDRDQKUlhJeXlESjdQU2pMa2FTbktwTzFMNmNhQ1BzVUUyVWxxVHZvUUxjOE9lMGgvaldPUEFnTElneUFkZngvL2QvaQo0M0xMd
FN2S0wxTVY5OHA5bFVIc3QzWHBBZ01CQUFHZ0FEQU5CZ2txaGtpRzl3MEJBUXNGQUFPQ0FRRUFQT3R3CkdpdFhHcUo0cXA5S2Raa3ozZjRtL3VwK1VoWi83WExzeU9xS1BmanlSbUJiZzFiY3RXMWtFQlpZVUt1ZkJxSmoKd
EI4UFdyZzdOOHprWFlZWng5SUZ5bTJ6eFQ1eXZyaC94ZnFaRXVHQ1JLbHFpYmxPdDU4WXJXQ0pQemZWQlU4RgpraThMYVhiY2RiSFIva3lscEYxSkNsTCtyWExEQnVGbDE5ekU0Z3BPZENNSXg3UllleWFneXZ6TXBieEdQT
kNICk10K2tFS2ppaXZOem80T2p6V0lQYWVocExwS3N4SDlVUE1FLzhOU2szaExYclBkdnJIa1JnVGxYU3N6QWI0UHoKVnl2LytpdXRTRnNKQkJXZEtkcHg0OSt4SXRMYnUyaThHclkyaUNuMlVoZEt6Z1pIWVI5bUFOYmJ4Z
FBjZk83agpXYjVBVTN1NFhYOW15aTEyT1E9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
```

```bash
echo "L********" |base64 --decode

```
```bash
kubectl config --kubeconfig=config-demo set-credentials nyarlagadda --client-certificate=nyarlagadda.crt --client-key=nyarlagadda.key
```

```bash
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: nyarlagadda
spec:
  groups:
  - system:authenticated
  usages:
  - digital signature
  - key encipherment
  - server auth
  request: 
```
```bash

https://8gwifi.org/docs/kube-rbac.jsp
```
```bash
kubectl config set-credentials user2 --client-certificate=user2.crt --client-key=user2.key
kubectl config set-credentials nyarlagadda --client-certificate=/home/ynraju4/nyarlagadda.crt --client-key=/home/ynraju4/nyarlagadda.key
kubectl config set-context test --cluster=hello-world.k8s.local --namespace=kube-system --user=nyarlagadda
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
