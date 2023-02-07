# Overview
Using Windows Server containers on GKE enables you to take advantage of the benefits of Kubernetes: agility, speed of deployment and simplified management of your Windows Server applications. You can run your Windows Server and Linux containers side by side in the same cluster, which allows for a central management plane for both container platforms. Microsoft Hyper-V containers are not currently supported.

# Creating a cluster using Windows Server node pools
To create this cluster you need to complete the following tasks:
1. Choose your Windows Server node image.
2. Update and configure gcloud.
3. Create a cluster and node pools.
4. Get kubectl credentials.
5. Wait for cluster initialization.

# Hands-on
1. Create a cluster with a windows node pool and a linux node pool. Please replace "your-projectid" as your project id, the following command has 4 places need to be modified and we use Windows Server version 2019 (LTSC) as the container node image.

`Note : To run Windows Server containers, your cluster must have at least one Windows and one Linux node pool. You cannot create a cluster using only a Windows Server node pool.`

```
gcloud beta container --project "your-projectid" clusters create "windows-cluster" --zone "asia-east1-a" --no-enable-basic-auth --cluster-version "1.23.8-gke.1900" --release-channel "regular" --machine-type "n1-standard-2" --image-type "COS_CONTAINERD" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "3" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/your-projectid/global/networks/default" --subnetwork "projects/your-projectid/regions/asia-east1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "asia-east1-a" && gcloud beta container --project "your-projectid" node-pools create "windows-pool" --cluster "windows-cluster" --zone "asia-east1-a" --machine-type "n1-standard-2" --image-type "WINDOWS_LTSC_CONTAINERD" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --num-nodes "3" --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --max-pods-per-node "110" --node-locations "asia-east1-a"
```

2. Connect to the cluster, please replace "your-projectid" as your project id.
```
gcloud container clusters get-credentials windows-cluster --zone asia-east1-a --project your-projectid
```
3. Delpoy a windows server application
Windows Server nodes are tainted with the following key-value pair: node.kubernetes.io/os=windows
```
kubectl apply -f iis.yaml
```
### iis.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iis
  labels:
    app: iis
spec:
  replicas: 1
  selector:
    matchLabels:
      app: iis
  template:
    metadata:
      labels:
        app: iis
    spec:
      nodeSelector:
        kubernetes.io/os: windows
      containers:
      - name: iis-server
        image: mcr.microsoft.com/windows/servercore/iis
        ports:
        - containerPort: 80
```
4. Expose the Deployment.

```
kubectl expose deployment iis \
    --type=LoadBalancer \
    --name=iis
```

5. Verify that the Pod is running.

```
kubectl get pods
kubectl get service iis
```

6. Now, you can use your browser to open http://EXTERNAL_IP to see the IIS web page.

7. Clean up.
```
kubectl delete deploy iis
```
If you have created additional GKE clusters, use the following command to delete them.
```
gcloud container clusters delete CLUSTER_NAME --zone=asia-east1-a
```

## Source
- https://cloud.google.com/kubernetes-engine/docs/concepts/windows-server-gke
- https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-cluster-windows
- https://cloud.google.com/kubernetes-engine/docs/how-to/deploying-windows-app