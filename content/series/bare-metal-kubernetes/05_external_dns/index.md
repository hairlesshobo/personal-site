+++
title = 'Bare-metal Kubernetes - Part 5: External DNS'
slug = '05-external-dns'
date = "2024-07-28T14:00:00"
tags = ['home lab', 'kubernetes', 'self hosted']
series = ["Bare-metal Kubernetes"]
keywords = ['home lab', 'kubernetes', 'self hosted']
summary = 'How to enable automatic external DNS update support on my bare-metal Kubernetes cluster'
draft = true
+++

## Introduction

In the [previous article](/series/bare-metal-kubernetes/04-load-balancer), I setup MetalLB to enable load balancer support on the bare-metal kube cluster. 

This article builds on that by enabling Kubernetes to automatically add DNS entries to an external DNS server, in this case, [PowerDNS](https://www.powerdns.com/). This is possible using the [external-dns](https://github.com/kubernetes-sigs/external-dns) tool.

## Configure PowerDNS

The first thing you have to do is ensure that the API is enabled and an API key is set for your PowerDNS authoritative server. The official docs can be found [here](https://doc.powerdns.com/authoritative/http-api/index.html) but I will sum it up below as well.

For the rest of this article, we're going to assume that your DNS server IP is 192.168.1.42 because you know - [42](https://simple.wikipedia.org/wiki/42_(answer)).

1. Modify `pdns.conf` and ensure the following items are set
```makefile
# enable the powerdns internal webserver
webserver=yes

# IP address to bind to. default is 127.0.0.1 which won't work. 
webserver-address=192.168.1.42

# Port for the webserver to listen on. Change to anything you like, doesn't have to be 8081 this is just the default
webserver-port=8081

# This is important, this lists the CIDRs that are allowed to access the webserver. This must be configured to 
# allow traffic from the kubernetes cluster nodes. Using a typical 192.168.1.x range. Could do 0.0.0.0/0 if you 
# wanted but I wouldn't recommend it. Separate multiple with comma or whitespace
webserver-allow-from=192.168.1.0/24

# enable the API
api=yes

# Set the API key - generate something random and longish here
api-key=<changeme>
```

2. Restart powerdns - I'm honestly guessing on this one as I run PowerDNS as docker images using `docker-compose` at the moment, but I assume most people aren't.

```bash
systemctl restart powerdns
```

3. After it restarts, its probably best to make sure that the DNS server is running by manually querying it.


## Install and configure external-dns

There are a few files we have to create to setup the `external-dns` tool, but thankfully its not a very difficult process. I prefer to have external-dns in its own namespace. If you rather have it in `default`, you would need to modify the files below to suit your needs.

1. Create the namespace file `external-dns-ns.yaml` and apply
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: external-dns
  labels:
    name: external-dns
---
```

```bash
kubectl apply -f external-dns-ns.yaml
```

2. Create the role needed for external-dns in file `external-dns-role.yaml` and apply
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: external-dns
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions","networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: external-dns
```

```bash
kubectl apply -f external-dns-role.yaml
```

3. Prepare the api key secret by base64 encoding it
```bash
echo -n "<your-pdns-api-key-here>" | base64
```

4. Create the kube secret with file `external-dns-secret.yaml` and apply
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: external-dns
  namespace: external-dns
type: Opaque
data:
  apiKey: <base64-encoded-secret>
```

```bash
kubectl apply -f external-dns-secret.yaml
```

5. Create the deployment as file `external-dns-deployment.yaml` and apply
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: external-dns
  labels:
    kubernetes.io/cluster-service: "true"
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      # This section and the tolerations section all it to be scheduled on control planes. 
      # If you prefer it run on standard nodes remove these two sections. Additionally, if 
      # you removed the control plane taint, as outlined in the the second article of the 
      # series, this is irrelevant and can be removed
      nodeSelector:
        node-role.kubernetes.io/control-plane:
      tolerations:
      - effect: NoExecute
        operator: Exists
      - effect: NoSchedule
        operator: Exists
      containers:
      - name: external-dns
        image: registry.k8s.io/external-dns/external-dns:v0.14.2
        args:
        - --source=service # or ingress or both
        - --service-type-filter=LoadBalancer # this will only create dns entries for serviecs of type LoadBalancer
        - --provider=pdns
        - --pdns-server=http://192.168.1.42:8081
        - --txt-owner-id=kubernetes # this can be anything you want, but make sure it doesn't change once you set it initially
        - --domain-filter=kube.internal.example.com # will make ExternalDNS see only the zones matching provided domain; omit to process all available zones in PowerDNS
        - --log-level=debug # might want to change this to info one you know its working
        - --interval=30s
        env:
        - name: EXTERNAL_DNS_PDNS_API_KEY
          valueFrom:
            secretKeyRef:
              name: external-dns
              key: apiKey
```

```bash
kubectl apply -f external-dns-deployment.yaml
```

## Testing the system

Coming soon..
