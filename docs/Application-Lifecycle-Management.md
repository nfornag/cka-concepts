## Application Lifecycle Management 8%

### 1. Deploying an Application, Rolling Updates, and Rollbacks 

We already know Kubernetes will run pods and deployments, but what happens when you need to update or change the version of your application running inside of the Kubernetes cluster? That’s where rolling updates come in, allowing you to update the app image with zero downtime. In this lesson, we’ll go over a rolling update, how to roll back, and how to pause the update if things aren’t going well.

The YAML for a deployment:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubeserve
  template:
    metadata:
      name: kubeserve
      labels:
        app: kubeserve
    spec:
      containers:
      - image: linuxacademycontent/kubeserve:v1
        name: app
```

Create a deployment with a record (for rollbacks):

```bash
kubectl create -f kubeserve-deployment.yaml --record
```

Check the status of the rollout:

```bash
kubectl rollout status deployments kubeserve

```
View the ReplicaSets in your cluster:

```bash
kubectl get replicasets
```
Scale up your deployment by adding more replicas:

```bash
kubectl scale deployment kubeserve --replicas=5
```
Expose the deployment and provide it a service:

```bash
kubectl expose deployment kubeserve --port 80 --target-port 80 --type NodePort
```
Set the minReadySeconds attribute to your deployment:

```bash
kubectl patch deployment kubeserve -p '{"spec": {"minReadySeconds": 10}}'
```
Use kubectl apply to update a deployment:

```bash
kubectl apply -f kubeserve-deployment.yaml
```
Use kubectl replace to replace an existing deployment:

```bash
kubectl replace -f kubeserve-deployment.yaml
```
Run this curl look while the update happens:

```bash
while true; do curl http://10.105.31.119; done
```
Perform the rolling update:

```bash
kubectl set image deployments/kubeserve app=linuxacademycontent/kubeserve:v2 --v 6
```
```bash
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1
kubectl rollout status deployment/myapp-deployment
kubectl rollout history deployment/myapp-deployment
kubectl rollout undo deployment/myapp-deployment
kubectl describe deployment frontend |grep -i StrategyType:
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v3
```

Describe a certain ReplicaSet:

```bash
kubectl describe replicasets kubeserve-[hash]
```
Apply the rolling update to version 3 (buggy):

```bash
kubectl set image deployment kubeserve app=linuxacademycontent/kubeserve:v3
```
Undo the rollout and roll back to the previous version:

```bash
kubectl rollout undo deployments kubeserve
```
```bash
Look at the rollout history:

```bash
kubectl rollout history deployment kubeserve
```
Roll back to a certain revision:

```bash
kubectl rollout undo deployment kubeserve --to-revision=2
```

Pause the rollout in the middle of a rolling update (canary release):


```bash
kubectl rollout pause deployment kubeserve
```
Resume the rollout after the rolling update looks good:

```bash
kubectl rollout resume deployment kubeserve
```
Helpful Links
Deployments https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
Creating a Deployment https://kubernetes.io/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/
Performing a Rolling Update https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/




### 2. Configuring an Application for High Availability and Scale 

Continuing from the last lesson, we will go through how Kubernetes will save you from EVER releasing code with bugs. Then, we will talk about ConfigMaps and secrets as a way to pass configuration data to your apps.

The YAML for a readiness probe:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 3
  selector:
    matchLabels:
      app: kubeserve
  minReadySeconds: 10
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
    type: RollingUpdate
  template:
    metadata:
      name: kubeserve
      labels:
        app: kubeserve
    spec:
      containers:
      - image: linuxacademycontent/kubeserve:v3
        name: app
        readinessProbe:
          periodSeconds: 1
          httpGet:
            path: /
            port: 80

```
Apply the readiness probe:

```bash
kubectl apply -f kubeserve-deployment-readiness.yaml
```
View the rollout status:

```bash
kubectl rollout status deployment kubeserve
```
Describe deployment:

```bash
kubectl describe deployment
```
Create a ConfigMap with two keys:

```bash
kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2
```
Get the YAML back out from the ConfigMap:

```bash
kubectl get configmap appconfig -o yaml
```
The YAML for the ConfigMap pod:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
  - name: app-container
    image: busybox:1.28
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    env:
    - name: MY_VAR
      valueFrom:
        configMapKeyRef:
          name: appconfig
          key: key1
```
Create the pod that is passing the ConfigMap data:

```bash
kubectl apply -f configmap-pod.yaml
```
Get the logs from the pod displaying the value:

```bash
kubectl logs configmap-pod
```
The YAML for a pod that has a ConfigMap volume attached:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    volumeMounts:
      - name: configmapvolume
        mountPath: /etc/config
  volumes:
    - name: configmapvolume
      configMap:
        name: appconfig
```
* Using Configmap as environment 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

Create the ConfigMap volume pod:

```bash
kubectl apply -f configmap-volume-pod.yaml
```
Get the keys from the volume on the container:

```bash
kubectl exec configmap-volume-pod -- ls /etc/config
```
Get the values from the volume on the pod:

```bash
kubectl exec configmap-volume-pod -- cat /etc/config/key1
```
The YAML for a secret:

```bash
apiVersion: v1
kind: Secret
metadata:
  name: appsecret
stringData:
  cert: value
  key: value
```
Create the secret:

```bash
kubectl apply -f appsecret.yaml
```
The YAML for a pod that will use the secret:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
    env:
    - name: MY_CERT
      valueFrom:
        secretKeyRef:
          name: appsecret
          key: cert
```
Create the pod that has attached secret data:

* Create Secrets

```bash
kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
```

* Using secret as environment 

```bash
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

```bash
kubectl apply -f secret-pod.yaml
```
Open a shell and echo the environment variable:

```bash
kubectl exec -it secret-pod -- sh
echo $MY_CERT
```
The YAML for a pod that will access the secret from a volume:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app-container
    image: busybox
    command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
    volumeMounts:
      - name: secretvolume
        mountPath: /etc/certs
  volumes:
    - name: secretvolume
      secret:
        secretName: appsecret
```
Create the pod with volume attached with secrets:

```bash
kubectl apply -f secret-volume-pod.yaml
```
Get the keys from the volume mounted to the container with the secrets:

```bash
kubectl exec secret-volume-pod -- ls /etc/certs
```
Helpful Links
Scaling Your Application https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#scaling-your-application
Configure Pod ConfigMaps https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
Secrets https://kubernetes.io/docs/concepts/configuration/secret/

### 3. Creating a Self-Healing Application 

In this lesson, we’ll go through the power of ReplicaSets, which make your application self-healing by replicating pods and moving them around and spinning them up when nodes fail. We’ll also talk about StatefulSets and the benefit they provide.

The YAML for a ReplicaSet:

```bash
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myreplicaset
  labels:
    app: app
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: main
        image: linuxacademycontent/kubeserve
```
Create the ReplicaSet:

```bash
kubectl apply -f replicaset.yaml
```
The YAML for a pod with the same label as a ReplicaSet:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: main
    image: linuxacademycontent/kubeserve
```
Create the pod with the same label:

```bash
kubectl apply -f pod-replica.yaml
```
Watch the pod get terminated:

```bash
kubectl get pods -w 
```
The YAML for a StatefulSet:

```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Create the StatefulSet:

```bash
kubectl apply -f statefulset.yaml
```
View all StatefulSets in the cluster:

```bash
kubectl get statefulsets
```
Describe the StatefulSets:

```bash
kubectl describe statefulsets
```
Helpful Links
ReplicaSet 
StatefulSets
