# Exposing applications using services

The idea of a Service is to group a set of Pod endpoints into a single resource. You can configure various ways to access the grouping. By default, you get a stable cluster IP address that clients inside the cluster can use to contact Pods in the Service. A client sends a request to the stable IP address, and the request is routed to one of the Pods in the Service.

There are five types of Services:
- ClusterIP: Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service.
- NodePort: Exposes the Service on each Node's IP at a static port (the NodePort).
- LoadBalancer: Exposes the Service externally using a cloud provider's load balancer.
- ExternalName: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. No proxying of any kind is set up.


Autopilot clusters are public by default. If you opt for a private Autopilot cluster, you must configure Cloud NAT to make outbound internet connections, for example pulling images from DockerHub.

# Using kubectl expose to create a Service

As an alternative to writing a Service manifest, you can create a Service by using kubectl expose to expose a Deployment.

# Hands-On

1. Create and deply a nginx deployment.
```
$ vim my-nginx.yaml
$ kubectl create ns my-nginx
$ kubectl -n my-nginx apply -f my-nginx.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-nginx
  namespace: my-nginx
spec:
  selector:
    matchLabels:
      run: my-nginx
  replicas: 2
  template:
    metadata:
      labels:
        run: my-nginx
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80
```
2. Check your nginx deployment is up and running.
You can observe that nginx replica has two different IP addresses.
```
$ kubectl -n my-nginx get pods -o wide
```

3. create a Service for your nginx replicas with kubectl expose.

A Kubernetes Service is an abstraction which defines a logical set of Pods running somewhere in your cluster, that all provide the same functionality. When created, each Service is assigned a unique IP address called clusterIP. This address is tied to the lifespan of the Service, and will not change while the Service is alive. Pods can be configured to talk to the Service, and know that communication to the Service will be automatically load-balanced out to some pod that is a member of the Service.
```
$ kubectl -n my-nginx expose deployment/my-nginx
$ kubectl -n my-nginx get svc my-nginx
$ kubectl -n my-nginx describe svc my-nginx
```

As mentioned previously, a Service is backed by a group of Pods. These Pods are exposed through endpoints. The Service’s selector will be evaluated continuously and the results will be POSTed to an Endpoints object also named my-nginx. When a Pod dies, it is automatically removed from the endpoints, and new Pods matching the Service’s selector will automatically get added to the endpoints.

4. Currently the Service does not have an External IP, so let’s now patch the Service to use a cloud load balancer, by updating the type of the my-nginx Service from ClusterIP to LoadBalancer:

```
$ kubectl -n my-nginx patch svc my-nginx -p '{"spec": {"type": "LoadBalancer"}}'
$ kubectl -n my-nginx get svc my-nginx
```

5. Use kubectl describe service my-nginx to see LoadBalancer Ingress.
```
$ kubectl -n my-nginx describe service my-nginx | grep Ingress
```

## Source
- https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps
- https://cloud.google.com/kubernetes-engine/docs/how-to/exposing-apps#using_kubectl_expose_to_create_a_service
- https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
