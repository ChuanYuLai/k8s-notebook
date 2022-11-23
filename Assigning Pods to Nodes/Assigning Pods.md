# Assigning Pods to Nodes

You can constrain a Pod so that it is restricted to run on particular node(s), or to prefer to run on particular nodes. There are several ways to do this and the recommended approaches all use label selectors to facilitate the selection.

However, there are some circumstances where you may want to control which node the Pod deploys to, for example, to ensure that a Pod ends up on a node with an SSD attached to it, or to co-locate Pods from two different services that communicate a lot into the same availability zone.

## nodeSelector
nodeSelector is the simplest recommended form of node selection constraint. You can add the nodeSelector field to your Pod specification and specify the node labels you want the target node to have. Kubernetes only schedules the Pod onto nodes that have each of the labels you specify.

## Affinity and anti-affinity
Node affinity is conceptually similar to nodeSelector, allowing you to constrain which nodes your Pod can be scheduled on based on node labels. There are two types of node affinity:

- requiredDuringSchedulingIgnoredDuringExecution: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
- preferredDuringSchedulingIgnoredDuringExecution: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

## Hands-on Node Selector
1. Check the lable of your nodes.
```
$ kubectl get nodes --show-labels -o wide
```
2. Label one of the nodes with disk_type=ssd.
```
kubectl label node/node_name disk_type=ssd
```
3. Check if the label is valid.
```
$ kubectl get nodes --show-labels -o wide
```
4. Deploy an nginx pod, specifying nodeSelector equal to disk_type=ssd.
```
$ vim nodeselector.yaml
$ kubectl apply -f nodeselector.yaml
```
### nodeselector.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disk_type: ssd
 ```

5. Check which node the nginx pod is assigned to
```
$ kubectl get pod -o wide
```

## Hands-on Node Affinity
1. Create a redis deployment and a nginx web server deployment, each deployment have three pods.
```
$ vim redis.yaml
$ vim webserver.yaml
```
### redis.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-cache
spec:
  selector:
    matchLabels:
      app: store
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```
### webserver.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-server
spec:
  selector:
    matchLabels:
      app: web-store
  replicas: 3
  template:
    metadata:
      labels:
        app: web-store
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - web-store
            topologyKey: "kubernetes.io/hostname"
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: web-app
        image: nginx:1.16-alpine
```

2. Deply and check those deployment
```
$ kubectl apply -f redis.yaml
$ kubectl apply -f webserver.yaml
$ kubectl get deployment
```

3. You can see that the k8s system has assigned the web app and redis to different worker nodes in a one-to-one pair.
```
$ kubectl get pod -o wide
```
As you can see from the example above, if you don't want the pods to be assigned to the same node, just set podAntiAffinity and topologyKey: "kubernetes.io/hostname"



## Source
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#more-practical-use-cases
