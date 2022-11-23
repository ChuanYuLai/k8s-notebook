# Background
By nature, Pods are ephemeral. This means that GKE destroys the state and value stored in a Pod when it is deleted, evicted, or rescheduled.

As an application operator, you may want to maintain stateful workloads. Examples of such workloads include applications that process WordPress articles, messaging apps, and apps that process machine learning operations.

By using Filestore on GKE, you can perform the following operations:
- Deploy stateful workloads that are scalable.
- Enable multiple Pods to have ReadWriteMany as its accessMode, so that multiple Pods can read and write at the same time to the same storage.
- Set up GKE to mount volumes into multiple Pods simultaneously.
- Persist storage when Pods are removed.
- Enable Pods to share data and easily scale.

Deploy a stateful workload with Filestore:

1. Create a Filestore instance (Basic type)

    Filestore instances are fully managed NFS file servers on Google Cloud for use with applications running on Compute Engine virtual machines instances or Google Kubernetes Engine clusters.
    ```
    $ gcloud filestore instances create test-nfs --tier=basic-hdd --file-share=name=test,capacity=1TB --network=name=default --zone=asia-east1-b 
    ```
    `Note: The minimum storage volume of Filestore must be 1 TB.`

2. Create a GKE cluster
    ```
    $ gcloud beta container --project "test-nfs" clusters create "nfs-cluster" --zone "asia-east1-a" --no-enable-basic-auth --cluster-version "1.23.8-gke.1900" --release-channel "None" --machine-type "e2-medium" --image-type "COS_CONTAINERD" --disk-type "pd-standard" --disk-size "100" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "3" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/test-nfs/global/networks/default" --subnetwork "projects/test-nfs/regions/asia-east1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --node-locations "asia-east1-a"
    ```

3. Create a Persistent Volume and Persistent Volume Claim
    ### nfs-pv.yaml
    ```
    apiVersion: v1
    kind: PersistentVolume
    metadata:
    name: filestore-nfs-pv
    spec:
    capacity:
        storage: 20Gi
    accessModes:
    - ReadWriteMany
    nfs:
        path: /test
        server: 10.x.x.x
    ```
    Using kubectl command to create the PV.
    ```
    $ kubectl create -f nfs-pv.yaml
    ```
    ### nfs-pvc.yaml
    ```
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
    name: filestore-nfs-pvc
    spec:
    accessModes:
    - ReadWriteMany
    storageClassName: ""
    volumeName: filestore-nfs-pv
    resources:
        requests:
        storage: 20Gi
    ```
    Using kubectl command to create the PVC.
    ```
    $ kubectl create -f nfs-pvc.yaml
    ```
    Check the PV and PVC.
    ```
    $ kubectl get pv
    $ kubectl get pvc
    ```
4. Create a Nginx Deployment to mount the created PVC
    ### sample-nginx.yaml
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-deployment
    spec:
    selector:
        matchLabels:
        app: nginx
    replicas: 1
    template:
        metadata:
        labels:
            app: nginx
        spec:
        containers:
        - name: nginx
            image: nginx:1.14.2
            volumeMounts:
            - mountPath: /mnt/shared-files
            name: nfs-pvc
            ports:
            - containerPort: 80
        volumes:
        - name:  nfs-pvc
            persistentVolumeClaim:
            claimName: filestore-nfs-pvc
            readOnly: false
    ```
    ```
    $ kubectl create -f sample-nginx.yaml
    $ kubectl get deployment
    ```


5. Go into the Nginx pod, create a test.txt file, then delete the Deployment and test if the PV has been mounted successfully.

## Source
- https://cloud.google.com/kubernetes-engine/docs/tutorials/stateful-workload
