# Why use StatefulSets?

You can use StatefulSets to deploy stateful applications and clustered applications that save data to persistent storage, such as Compute Engine persistent disks. StatefulSets are suitable for deploying Kafka, MySQL, Redis, ZooKeeper, and other applications needing unique, persistent identities and stable hostnames.

StatefulSets represent a set of Pods with unique, persistent identities, and stable hostnames that GKE maintains regardless of where they are scheduled. StatefulSets use a Pod template, which contains a specification for its Pods. The state information and other resilient data for any given StatefulSet Pod is maintained in persistent volumes associated with each Pod in the StatefulSet. StatefulSet Pods can be restarted at any time.

For stateless applications, use Deployments.

## Requesting persistent storage in a StatefulSet

Typically, you must create PersistentVolumeClaim objects in addition to creating the Pod. However, StatefulSet objects include a volumeClaimTemplates array, which automatically generates the PersistentVolumeClaim objects. Each StatefulSet replica gets its own PersistentVolumeClaim object.

## Hands-on
Once created, the StatefulSet ensures that the desired number of Pods are running and available at all times. The StatefulSet automatically replaces Pods that fail or are evicted from their nodes, and automatically associates new Pods with the storage resources, resource requests and limits, and other configurations defined in the StatefulSet's Pod specification.
1. Create a nginx service
```
$ kubectl apply -f nginx-service.yaml
```
### nginx-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
```
2. Create a statefulset
```
$ kubectl apply -f statefulset.yaml
```

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # Label selector that determines which Pods belong to the StatefulSet
                 # Must match spec: template: metadata: labels
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx # Pod template's label selector
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
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

>**Note**: To prevent data loss, PersistentVolumes and PersistentVolumeClaims are not deleted when a StatefulSet is deleted. You must manually delete these objects using kubectl delete pv and kubectl delete pvc.



## Source
- https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset
- https://cloud.google.com/kubernetes-engine/docs/how-to/stateful-apps#requesting_persistent_storage_in_a_statefulset
