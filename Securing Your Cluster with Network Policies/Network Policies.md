# GKE's network policy

You can use GKE's network policy enforcement to control the communication between your cluster's Pods and Services. You define a network policy by using the Kubernetes Network Policy API to create Pod-level firewall rules. These firewall rules determine which Pods and Services can access one another inside your cluster.

Network policies allow you to limit connections between Pods. Therefore, using network policies provide better security by reducing the compromise radius.

# Overhead, limitations, and caveats

- Enabling network policy enforcement consumes additional resources in nodes. Specifically, it increases the memory footprint of the kube-system process by approximately 128 MB, and requires approximately 300 millicores of CPU.

- Enabling network policy enforcement requires that your nodes be re-created. If your cluster has an active maintenance window, your nodes are not automatically re-created until the next maintenance window. If you prefer, you can manually upgrade your cluster at any time.

- The recommended minimum cluster size to run network policy enforcement is three e2-medium instances.
Network policy is not supported for clusters whose nodes are f1-micro or g1-small instances, as the resource requirements are too high for instances of that size.

`Note that the network policies determine whether a connection is allowed, and they do not offer higher level features like authorization or secure transport (like SSL/TLS).`

# Hands-On

1. Create a network policy namespace “np-ns”
```
$ kubectl create namespace np-ns
```

2. Create a nginx pod “nginx-network-policy-test”, and create a pod “nginx-can-access” that allow access nginx pod and a pod “nginx-cannot-access” that don't allow access nginx pod
```
$ vim nginx-network-policy-test
$ vim nginx-can-access
$ vim nginx-cannot-access
```

3. Deploy those pods
```
$ kubectl create -f nginx-network-policy-test.yaml
$ kubectl create -f nginx-can-access.yaml
$ kubectl create -f nginx-cannot-access.yaml
```

### nginx-network-policy-test.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-network-policy-test
  labels:
    app: nginx-network-policy-test
  namespace: np-ns
spec:
  containers:
  - name: nginx
    image: nginx:1.14-alpine
    ports:
    - containerPort: 80
```

nginx-can-access.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-can-access
  labels:
    priv: can-access
  namespace: np-ns
spec:
  containers:
  - name: nettools
    image: travelping/nettools
    command: ['sh', '-c', 'sleep 3600']
```

nginx-cannot-access.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-cannot-access
  namespace: np-ns
spec:
  containers:
  - name: nettools
    image: travelping/nettools
    command: ['sh', '-c', 'sleep 3600']
```

4. Check the result
```
$ kubectl get pod -o wide -n np-ns
$ kubectl describe ns/np-ns
```

5. Before applying the network policy, both pods can access the nginx pod normally
```
$ kubectl -n np-ns exec nginx-can-access -- /bin/sh -c "ping -c 2 nginx-network-policy-test-ip"
$ kubectl -n np-ns exec nginx-cannot-access -- /bin/sh -c "ping -c 2 nginx-network-policy-test-ip"
```

6. Create and deploy a network policy
```
$ vim deny-nginx-access.yaml
$ kubectl create -f deny-nginx-access.yaml
```

deny-nginx-access.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-nginx-access
  namespace: np-ns
spec:
  podSelector:
    matchLabels:
      app: nginx-network-policy-test
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              priv: can-access
```

7. Check the network policy result
```
$ kubectl get netpol -n np-ns
```

8. After applying the network policy, only nginx-can-access allow access nginx pod
```
$ kubectl -n np-ns exec nginx-can-access -- /bin/sh -c "ping -c 2 nginx-network-policy-test-ip"
$ kubectl -n np-ns exec nginx-cannot-access -- /bin/sh -c "ping -c 2 nginx-network-policy-test-ip"
```

9. Clean Up
```
$ kubectl delete ns np-ns
$ kubectl delete pod nginx-can-access -n np-ns
$ kubectl delete pod nginx-cannot-access -n np-ns
$ kubectl delete pod nginx-network-policy-test -n np-ns
$ kubectl -n np-ns delete netpol deny-nginx-access
```


## Source
- https://cloud.google.com/kubernetes-engine/docs/how-to/network-policy
- https://cloud.google.com/kubernetes-engine/docs/tutorials/network-policy
  
