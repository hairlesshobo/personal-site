+++
title = 'Bare-metal Kubernetes - Part 3: Networking'
slug = 'kube-cluster-networking'
date = "2024-07-27"
tags = ['home lab', 'kubernetes', 'self hosted']
series = ["Bare-metal Kubernetes"]
keywords = ['home lab', 'kubernetes', 'self hosted']
summary = 'How to enable networking with BGP support via OPNsense on my bare-meteal Kubernetes cluster'
draft = true
+++

## Introduction

In the [previous article](/posts/kube-cluster-bootstrap), I prepared my nodes, created the new kube cluster and added all my nodes to the cluster.

This article will outline the process that I followed to setup Calico, the network provider, known in Kubernetes land as "Container Network Interface". 

## Credits

So I want to first start off by saying that this portion was largely possible for me thanks to an [excellent article by tizbit](https://tyzbit.blog/configuring-bgp-with-calico-on-k8s-and-opnsense). The only reason I am writing my own article about it is because I want to make sure and cover the Calico installation process prior to setting up the BGP routing. 

## Overview

When you are configuring the CNI for Kubernetes, you will need to define a few IP ranges. In [part 2](/posts/kube-cluster-bootstrap) we defined two of those network ranges already, the pod network CIDR block and the service CIDR block. Additionally, we need to define service external CIDR block and load balancer CIDR block. I have chosen the following CIDR blocks, feel free to change them but make sure you take note of the ranges you choose as you will need them below.

| Use                      | CIDR         |
|--------------------------|--------------|
| Pod network              | 10.29.0.0/16 |
| Service - External       | 10.30.0.0/16 |
| Service - Load Balancer  | 10.31.0.0/16 |
| Service - Cluster        | 10.32.0.0/12 |


## Install the Calico CNI

The installation process is pretty well outlined [here](https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises) but I will sum up the steps here too, just to be thorough.

1. First you need to install the Calico operator, which apparently does all the heavy lifting automatically. Don't ask me to fully explain what all it does as I haven't read too deep into it yet.

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/tigera-operator.yaml
```

2. Download a copy of the custome resources needed to configure Calico.

```bash
curl https://raw.githubusercontent.com/projectcalico/calico/v3.28.0/manifests/custom-resources.yaml -O
```

3. Using your favorite text edit (I suggest `vim`), edit `custom-resources.yaml`. You need to edit `cidr` under the first `spec.calicoNetwork.ipPools` entry so that it matches the "Pod network" selected above, in my case `10.29.0.0/16`. This is how my file looks. You can likely use it as-is, but its probably better to pull a fresh copy using step 2 and edit, in case the default changes from Calico.

```yaml
# This section includes base Calico installation configuration.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    ipPools:
    - name: default-ipv4-ippool
      blockSize: 26
      cidr: 10.29.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()

---

# This section configures the Calico API server.
# For more information, see: https://docs.tigera.io/calico/latest/reference/installation/api#operator.tigera.io/v1.APIServer
apiVersion: operator.tigera.io/v1
kind: APIServer
metadata:
  name: default
spec: {}
```

4. Create the Calico configuration

```bash
kubectl create -f custom-resources.yaml
```

5. Wait for Calico to fully start up

```bash
watch kubectl get pods -n calico-system
```

You should see something simmilar to this:

```plain
NAME                                       READY   STATUS    RESTARTS       AGE
calico-kube-controllers-7f7795754c-qq9px   1/1     Running   1 (42h ago)    7d22h
calico-node-t66tp                          1/1     Running   1 (42h ago)    7d22h
calico-typha-7b598d75c9-8rtdd              1/1     Running   1 (42h ago)    7d22h
csi-node-driver-7bchz                      2/2     Running   60 (37h ago)   7d22h
```

Once all pods are running, hit `Ctrl+C` to stop the watch command.

## Configure BGP routing

I highly recommend you take a few minutes to read through [tizbit's article](https://tyzbit.blog/configuring-bgp-with-calico-on-k8s-and-opnsense) so that you can get an explanation of the `asNumber` field below. I don't want to go into it here, but its useful information to know.

1. Certain calico configuration must be done with the `calicoctl` tool. Install it first, as root (or use sudo)
```bash
cd /usr/local/sbin
curl -L https://github.com/projectcalico/calico/releases/download/v3.28.0/calicoctl-linux-amd64 -o calicoctl
chmod +x ./calicoctl
```

2. Create a `calico-bgp-config.yaml` file with the following contents. Be sure to edit the `spec.peerIP` toward the bottom to the internal IP address of the OPNsense router. Also, be sure to edit the three provided CIDR blocks if you chose a different set than I do.

```yaml
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  asNumber: 64513
  nodeToNodeMeshEnabled: true
  serviceExternalIPs:
    - cidr: 10.30.0.0/16
  serviceLoadBalancerIPs:
    - cidr: 10.31.0.0/16
  serviceClusterIPs:
    - cidr: 10.32.0.0/12
---
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: router
spec:
  peerIP: 192.168.1.1
  asNumber: 64512
  keepOriginalNextHop: true
  maxRestartTime: 15m
---
```

3. Apply the configuration using `calicoctl`
```bash
calicoctl apply -f calico-bgp-config.yaml
```

4. For each node in your cluster, create the following config. Be sure to oupdate the `node` line for the names of your nodes. Hint: You can combine them all into one file by separating them with `---`, for example:

```yaml
apiVersion: projectcalico.org/v3
kind: CalicoNodeStatus
metadata:
  name: donkey
spec:
  classes:
    - Agent
    - BGP
    - Routes
  node: donkey
  updatePeriodSeconds: 10
---
apiVersion: projectcalico.org/v3
kind: CalicoNodeStatus
metadata:
  name: donkey
spec:
  classes:
    - Agent
    - BGP
    - Routes
  node: starfox
  updatePeriodSeconds: 10
---
apiVersion: projectcalico.org/v3
kind: CalicoNodeStatus
metadata:
  name: donkey
spec:
  classes:
    - Agent
    - BGP
    - Routes
  node: mule
  updatePeriodSeconds: 10
```

Still in progress...