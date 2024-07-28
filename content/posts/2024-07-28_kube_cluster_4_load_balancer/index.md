+++
title = 'Bare-metal Kubernetes - Part 4: Load Balancer'
slug = 'kube-cluster-load-balancer'
date = "2024-07-28T12:00:00"
tags = ['home lab', 'kubernetes', 'self hosted']
series = ["Bare-metal Kubernetes"]
keywords = ['home lab', 'kubernetes', 'self hosted']
summary = 'How to enable load balancer support on my bare-metal Kubernetes cluster'
draft = false
+++

## Introduction

In the [previous article](/posts/kube-cluster-networking), I setup Calico as the CNI and enabled BGP routing via OPNsense.

This article will outline the process that I followed to add load balancer support to my bare-metal kubernetes cluster. Typically, load balancer support is handled automatically in cloud-based environments such as AWS, Azure, GCP, etc by leveraging the cloud load balancers already available in the cloud provider. For example an EKS cluster in AWS would automatically provision ELB instances any time you request a load balancer in Kubernetes. In a bare-metal install, Kubernetes does not provide any mechanism for load balancers. This is where [MetalLB](https://metallb.io/) comes in to play. I will use MetalLB to provide load balancer support.

## Intalling and configuring MetalLB

1. Install MetalLB to your cluster using `kubectl`:
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.14.8/config/manifests/metallb-native.yaml
```

2. Configure MetalLB by creating the file `metallb-ip-address-pool.yaml` to with the following contents, making sure to adjust the IP range for load balancers if you chose to use a different range than I did.
```yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.31.0.0/16
```

3. Apply the config using `kubectl`
```bash
kubectl apply -f metallb-ip-address-pool.yaml
```

## Testing the load balancer

The easiest way to test the load balancer is to spin up an nginx deployment and service with a load balancer configured. This is super easy.

1. Create `nginx-test.yaml` with the following contents:
```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  labels:
    app: nginx-test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-test
  template:
    metadata:
      labels:
        app: nginx-test
    spec:
      containers:
      - image: nginx
        name: nginx-test
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-test
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx-test
  type: LoadBalancer
```

2. Apply the file with `kubectl`
```bash
kubectl apply -f nginx-test.yaml
```

3. Wait a few moments and run `kubectl get all`, you should see an output similar to this:
```plaintext
NAME                                                READY   STATUS    RESTARTS        AGE
pod/nginx-test-5774b4685c-l8v9j                     1/1     Running   1 (2d15h ago)   7d21h

NAME                                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes                     ClusterIP      10.32.0.1       <none>        443/TCP        8d
service/nginx-test                     LoadBalancer   10.37.137.56    10.31.0.1     80:32491/TCP   8d

NAME                              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-test                     1/1     1            1           7d21h

NAME                                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-test-5774b4685c                     1         1         1       7d21h
```

4. Make sure that the line towards the top that starts with `pod/nginx-test-` shows a status of `Running`, if so it means nginx successfully started. Next, look for the `service/nginx-test` line and under the `EXTERNAL-IP` column, you should see an IP address assigned to it. In a browser somewhere not on your kube cluster, try accessing that IP address. You should be presented with a default nginx homepage if all is working. 

## Summary

Believe it or not, that's all it takes to setup MetalLB on your Kube cluster. Any time you set `type: LoadBalancer` on a service, MetalLB will automatically create a load balancer for you.

In the [next article](/posts/kube-cluster-external-dns), I will show you how to have Kubernetes automatically create a DNS entry in PowerDNS any time you add a load balancer. 