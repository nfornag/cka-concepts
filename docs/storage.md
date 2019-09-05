
## Storage 7%

### Persistent Volumes 

In Kubernetes, pods are ephemeral. This creates a unique challenge with attaching storage directly to the filesystem of a container. Persistent Volumes are used to create an abstraction layer between the application and the underlying storage, making it easier for the storage to follow the pods as they are deleted, moved, and created within your Kubernetes cluster.

In the Google Cloud Engine, find the region your cluster is in:

```bash
gcloud container clusters list
```
Using Google Cloud, create a persistent disk in the same region as your cluster:

```bash
gcloud compute disks create --size=1GiB --zone=us-central1-a mongodb
```
The YAML for a pod that will use persistent disk:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mongodb 
spec:
  volumes:
  - name: mongodb-data
    gcePersistentDisk:
      pdName: mongodb
      fsType: ext4
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
```
Create the pod with disk attached and mounted:

```bash
kubectl apply -f mongodb-pod.yaml
```
See which node the pod landed on:

```bash
kubectl get pods -o wide
```
Connect to the mongodb shell:

```bash
kubectl exec -it mongodb mongo
```
Switch to the mystore database in the mongodb shell:

```bash
use mystore
```
Create a JSON document to insert into the database:

```bash
db.foo.insert({name:'foo'})
```
View the document you just created:

```bash
db.foo.find()
```
Exit from the mongodb shell:

```bash
exit
```
Delete the pod:

```bash
kubectl delete pod mongodb
```
Create a new pod with the same attached disk:

```bash
kubectl apply -f mongodb-pod.yaml
```
Check to see which node the pod landed on:

```bash
kubectl get pods -o wide
```
Drain the node (if the pod is on the same node as before):

```bash
kubectl drain [node_name] --ignore-daemonsets
```
Once the pod is on a different node, access the mongodb shell again:

```bash
kubectl exec -it mongodb mongo
```
Access the mystore database again:

```bash
use mystore
```
Find the document you created from before:

```bash
db.foo.find()
```
The YAML for a PersistentVolume object in Kubernetes:

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity: 
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```

Create the Persistent Volume resource:

```bash
kubectl apply -f mongodb-persistentvolume.yaml
```
View our Persistent Volumes:

```bash
kubectl get persistentvolumes
```
Helpful Links:
Persistent Volumes https://kubernetes.io/docs/concepts/storage/persistent-volumes/
Configure Persistent Volume Storage https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/



### Volume Access Modes 


Volume access modes are how you specify the access of a node to your Persistent Volume. There are three types of access modes: ReadWriteOnce, ReadOnlyMany, and ReadWriteMany. In this lesson, we will explain what each of these access modes means and two VERY IMPORTANT things to remember when using your Persistent Volumes with pods.

The YAML for a Persistent Volume:

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity: 
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```
View the Persistent Volumes in your cluster:

```bash
kubectl get pv
```
Helpful Links:
Access Modes https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes


### Persistent Volume Claims 

Persistent Volume Claims (PVCs) are a way for an application developer to request storage for the application without having to know where the underlying storage is. The claim is then bound to the Persistent Volume (PV), and it will not be released until the PVC is deleted. In this lesson, we will go through creating a PVC and accessing storage within our persistent disk.

The YAML for a PVC:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc 
spec:
  resources:
    requests:
      storage: 1Gi
  accessModes:
  - ReadWriteOnce
  storageClassName: ""
```
Create a PVC:

```bash
kubectl apply -f mongodb-pvc.yaml
```
View the PVC in the cluster:

```bash
kubectl get pvc
```
View the PV to ensure it’s bound:

```bash
kubectl get pv
```
The YAML for a pod that uses a PVC:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: mongodb 
spec:
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc
```
Create the pod with the attached storage:

```bash
kubectl apply -f mongo-pvc-pod.yaml
```
Access the mogodb shell:

```bash
kubectl exec -it mongodb mongo
```
Find the JSON document created in previous lessons:

```bash
db.foo.find()
```
Delete the mongodb pod:

```bash
kubectl delete pod mogodb
```
Delete the mongodb-pvc PVC:

```bash
kubectl delete pvc mongodb-pvc
```
Check the status of the PV:

```bash
kubectl get pv
```
The YAML for the PV to show its reclaim policy:

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv
spec:
  capacity: 
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
  persistentVolumeReclaimPolicy: Retain
  gcePersistentDisk:
    pdName: mongodb
    fsType: ext4
```
Helpful Links
PersistentVolumeClaims https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims
Create a PersistentVolumeClaim https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/#create-a-persistentvolumeclaim


### Storage Objects 

There’s an even easier way to provision storage in Kubernetes with StorageClass objects. Also, your storage is safe from data loss with the “Storage Object in Use Protection” feature, which ensures any pods using a Persistent Volume will not lose the data on the volume as long as it is actively mounted. We’ve been using Google Storage for this section, but there are many different volume types you can use in Kubernetes. In this lesson, we will talk about the hostPath volume and the empty directory volume type.

See the PV protection on your volume:

```bash
kubectl describe pv mongodb-pv
```
See the PVC protection for your claim:

```bash
kubectl describe pvc mongodb-pvc
```
Delete the PVC:

```bash
kubectl delete pvc mongodb-pvc
```
See that the PVC is terminated, but the volume is still attached to pod:

```bash
kubectl get pvc
```
Try to access the data, even though we just deleted the PVC:

```bash
kubectl exec -it mongodb mongo
use mystore
db.foo.find()
```
Delete the pod, which finally deletes the PVC:

```bash
kubectl delete pods mongodb
```
Show that the PVC is deleted:

```bash
kubectl get pvc
```
YAML for a StorageClass object:

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
Create the StorageClass type "fast":

```bash
kubectl apply -f sc-fast.yaml
```
Change the PVC to include the new StorageClass object:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc 
spec:
  storageClassName: fast
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```
Create the PVC with automatically provisioned storage:

```bash
kubectl apply -f mongodb-pvc.yaml
```
View the PVC with new StorageClass:

```bash
kubectl get pvc
```
View the newly provisioned storage:

```bash
kubectl get pv
```
The YAML for a hostPath PV:

```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-hostpath
spec:
  storageClassName: local-storage
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```
The YAML for a pod with an empty directory volume:

```bash
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - image: busybox
    name: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 3600; done"]
    volumeMounts:
    - mountPath: /tmp/storage
      name: vol
  volumes:
  - name: vol
    emptyDir: {}
```
Helpful Links:
Object in Use Protection https://kubernetes.io/docs/concepts/storage/persistent-volumes/#storage-object-in-use-protection
Volumes https://kubernetes.io/docs/concepts/storage/volumes/

### Applications with Persistent Storage 

In this lesson, we’ll wrap everything up in a nice little bow and create a deployment that will allow us to use our storage with our pods. This is to demonstrate how a real-world application would be deployed and used for storing data.

The YAML for our StorageClass object:

```bash
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```
The YAML for our PVC:

```bash
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: kubeserve-pvc 
spec:
  storageClassName: fast
  resources:
    requests:
      storage: 100Mi
  accessModes:
    - ReadWriteOnce
```
Create our StorageClass object:

```bash
kubectl apply -f storageclass-fast.yaml
```
View the StorageClass objects in your cluster:

```bash
kubectl get sc
```
Create our PVC:

```bash
kubectl apply -f kubeserve-pvc.yaml
```
View the PVC created in our cluster:

```bash
kubectl get pvc
```
View our automatically provisioned PV:

```bash
kubectl get pv
```
The YAML for our deployment:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubeserve
spec:
  replicas: 1
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
      - env:
        - name: app
          value: "1"
        image: linuxacademycontent/kubeserve:v1
        name: app
        volumeMounts:
        - mountPath: /data
          name: volume-data
      volumes:
      - name: volume-data
        persistentVolumeClaim:
          claimName: kubeserve-pvc
```
Create our deployment and attach the storage to the pods:

```bash
kubectl apply -f kubeserve-deployment.yaml
```
Check the status of the rollout:

```bash
kubectl rollout status deployments kubeserve
```
Check the pods have been created:

```bash
kubectl get pods
```
Connect to our pod and create a file on the PV:

```bash
kubectl exec -it [pod-name] -- touch /data/file1.txt
```
Connect to our pod and list the contents of the /data directory:

```bash
kubectl exec -it [pod-name] -- ls /data
```
```bash
::::::::::::::
dpvc.yml
::::::::::::::
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-pvc
spec:
  storageClassName: local-storage
  resources:
    requests:
      storage: 5Gi
  accessModes:
    - ReadWriteOnce
::::::::::::::
dpv.yml
::::::::::::::
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-pv
spec:
  capacity:
    storage: 5Gi
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/drupal-data"
::::::::::::::
dr.yml
::::::::::::::
apiVersion: apps/v1
kind: Deployment
metadata:
  name: drupal
  labels:
    app: drupal
spec:
  replicas: 1
  selector:
    matchLabels:
      app: drupal
  template:
    metadata:
      labels:
        app: drupal
    spec:
      initContainers:
      - name: init-sites-volume
        image: drupal:8.6
        command: ["/bin/bash", "-c"]
        args: ['cp -r /var/www/html/sites/ /data/; chown www-data:www-data /data/ -R']
        volumeMounts:
        - name: durapalinitvolume
          mountPath: /data
      containers:
      - name: drupal
        image: drupal:8.6
        volumeMounts:
        - name: modules
          mountPath: /var/www/html/modules
          subPath: modules
        - name: profiles
          mountPath: /var/www/html/profiles
          subPath: profiles
        - name: sites
          mountPath: /var/www/html/sites
          subPath: sites
        - name: themes
          mountPath: /var/www/html/themes
          subPath: themes
      volumes:
      - name: durapalinitvolume
        persistentVolumeClaim:
          claimName: drupal-pvc
      - name: modules
        emptyDir: {}
      - name: profiles
        emptyDir: {}
      - name: sites
        emptyDir: {}
      - name: themes
        emptyDir: {}
```
