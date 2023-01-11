# Introduction

Helm is a package manager and application management tool for Kubernetes that packages multiple Kubernetes resources into a single logical deployment unit called a Chart.

Helm helps you to:

- Achieve a simple (one command) and repeatable deployment
- Manage application dependency, using specific versions of other application and services
- Manage multiple deployment configurations: test, staging, production and others
- Execute post/pre deployment jobs during application deployment
- Update/rollback and test application deployments

## Install the Helm CLI

Before we can get started configuring Helm, we’ll need to first install the command line tools that you will interact with. To do this, run the following:

```
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

```

We can verify the version

```
helm version --short
```

Let’s configure our first Chart repository. Chart repositories are similar to APT or yum repositories that you might be familiar with on Linux, or Taps for Homebrew on macOS.

Download the stable repository so we have something to start with:
```
helm repo add stable https://charts.helm.sh/stable
```

Once this is installed, we will be able to list the charts you can install:
```
helm search repo stable
```

Finally, let’s configure Bash completion for the helm command:
```
helm completion bash >> ~/.bash_completion
. /etc/profile.d/bash_completion.sh
. ~/.bash_completion
source <(helm completion bash)
```

## Hands-On

1. Cloud Shell is preinstalled with Helm binary files (v3.9.3).

Check the version with “helm version”:
```
$ helm version
```

2. Setup the official ingress-nginx repository.
```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm repo update
```

3. Setup the Nginx Ingress Controller in nginx namespace.
```
$ kubectl create ns nginx
```

4. Execute the Nginx’s chart.
```
$ helm install nginx ingress-nginx/ingress-nginx --namespace nginx --set rbac.create=true --set controller.publishService.enabled=true
```

5. Check the external IP of our Nginx Ingress controller:
```
$ kubectl get svc -n nginx
```

6. Create a hello-app deployment and expose it:
```
$ kubectl create deployment hello-app --image=gcr.io/google-samples/hello-app:1.0

$ kubectl expose deployment hello-app --port 8080 --target-port 8080
```

7. Deploy the ingress controller:

hello-app-ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: shop-ingress-master
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - http:
      paths:
      - path: /helloworld
        pathType: Prefix
        backend:
          service:
            name: hello-app
            port:
              number: 8080
```
Delpoy apply -f hello-app-ingress.yaml.
```
$ kubectl apply -f hello-app-ingress.yaml
```
8. Now, we can visit the url below. 
```
http://external-ip/helloworld
```
You can get the external ip from below command.
```
$ kubectl get svc -n nginx
```
8. Clean up. 
```
$ kubectl delete deploy hello-app
```
You can get the external ip from below command.
```
$ helm uninstall nginx --namespace nginx
```


## Source
