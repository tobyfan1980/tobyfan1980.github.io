---
layout: post
title:  "K8s learning notes"
date:   2018-04-27 12:00:00 -0800
categories:    Cloud
tags:    K8s
---

## Concepts ##
### Namespace ###
Used to separate resources in cluster. User can set a context with specific namespace and switch to that context. Then, all the operations will be done in the namespace.

```
kubectl config set-context <context name> --namespace=<name space> --cluster=<cluster name> --user=<username>
kubectl config use-context <context name>
```
after this, we can only see the resources in this namespace.

### Label and selector ###
Labels are key-value pairs attached to a objects. They are used to organize and select subsets of objects.
Selector is used to query objects by labels. Two cases:
1. add constraint when deploying a resource. In the following example, the pod will be deployed in the Node with label {accelerator: nvidia-tesla-p100}
```
apiVersion: v1
kind: Pod
metadata:
  name: cuda-test
spec:
  containers:
    - name: cuda-test
      image: "k8s.gcr.io/cuda-vector-add:v0.1"
      resources:
        limits:
          nvidia.com/gpu: 1
  nodeSelector:
    accelerator: nvidia-tesla-p100
```

2. filter resources in kubectl cmdline with parameter '-l'
```
kubectl get pods -l environment=production
```

### Annotation ###
Same as label, it is key-value pairs, used to keep metadata of objects. Different from label, it cannot be used to select. 
When K8s system operates on an object, it can do something special controlled by annotations.

### Taint and Toleration ###
We can specify taint on Nodes, and Toleration on Pods, in order to determine if a Pod can be deployed on a Node. 
* add taint on Node
```
kubectl taint nodes node1 key=value:NoSchedule
```
* set toleration on Pod
```
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
```
What K8s does is to go over all taints on a Node, if the Pod has a matching toleration, mark the taint as ignored. After this, if there are
un-ignored taint with 'effect'=NoSchedulre' the Pod won't be assigned to that Node.

### Pod ###
Basic unit in K8s. It can have multiple containers, sharing single IP and communicate via localhost. They can share data via Volume in Pod. 

## Controllers ##
### ReplicateSet ###
manage a set of pod replication, ensure the same number of pod is running. 

### Deployment ###
higher-level concept that manages ReplicaSets and provides declarative updates to pods, which means K8s will handle rolling update for you, but for ReplicaSet which is imperative,
you will need to do it yourself.

### DaemonSet ###
Ensure a Pod is running on each Node, except the Nodes are excluded by nodeSelector (taint & toleration)
Differently, the Node is selected by the configuration before Pods are being deployed, other that by K8s scheduler. 

### Job ###
Creates one or more Pod to complete (successfully) a task in certain number of times. Use 'backoffLimit' to control the maximum failed runs. 
Use 'activeDeadlineSeconds' to control the maximum time the job runs. 
Pod will not be deleted even the job completes. Have to use 'kubectl delete ' to clean up. 
```
.spec.parallelism
```
Mutiple Pods run at the same time, but as long as one succeeds, no new Pods created. The rest pods keep running, but the job is consider
successful no matter their result. 

## Service ##
Service is used to abstract a set of Pods using label selector, so others can access the Pods via the service's name without worrying
about the change of the underneath Pods. Most of time service is used to expose a deployment of pods to external traffic. 
* ClusterIP: default mode. Service can be accessed only within cluster. 
* NodePort: a port on each Node will be allocated to identify the service. The pods can be accessed via <NodeIP>:<NodePORT>
* LoadBalancer: use external loadbalancer to deliver traffic to pos underneath a service
Headless service: set ClusterIP=None. Then service won't do load balance to the pods, it is done by user. 

## ConfigMap ##
Used to inject configuration information to containers. We can define ConfigMap object which contains data, and ref the object in Pod's config file
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.how: very
  special.type: charm
    
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: gcr.io/google_containers/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.how
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: special.type
      envFrom:
        - configMapRef:
            name: env-config
  restartPolicy: Never
```

## Storage ##
### Volume ###
We can load volume to Pod, and the volume is shared with containers inside. The volume's life cycle is related to Pod not containers.
Volume can be many types, e.g. emptyDir
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```
GCE Disk
```
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
  volumes:
  - name: test-volume
    # This GCE PD must already exist.
    gcePersistentDisk:
      pdName: my-data-disk
      fsType: ext4
```
### Persistant Volume ###
PV is a subsystem used to manage the volumes and allocate them to Pods. User can ask for PV by specifying persistant volume claim (PVC)
in the Pod configuration. The system will find matching PV and bind it to the Pod. 

PV in volume type NFS and volume mode Filesystem (can be block as well). 
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

PVC with storageClass and label selector
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    
```

Pods access storage by using the claim as a volume
```
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
