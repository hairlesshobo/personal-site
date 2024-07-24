+++
title = 'Bare metal Kubernetes Cluster: Bootstrap [WIP]'
date = "2024-07-24"
draft = false
tags = ['home lab']
keywords = ['Kubernetes Cluster Build']
summary = 'Steps I followed to bootstrap my new Kubernetes cluster'
+++

## Overview

This article will outline the process I followed to bootstrap a brand new Kubernetes cluster from scratch, and subsequently adding additional nodes to the cluster.

As my systems are all running vanilla Debian, the package manager commands will all be using `apt`. 

```warning
WARNING:

I am bad and use root for work like this instead of using sudo while working on personal systems, so you will need to `su -` to root before starting. For the record, I know this is bad practice and I only do this on personal servers. You should 100% use sudo on the commands below where appropriate instead.
```

## Prep The Node

This step is required for every node you add to the cluster.

Start off by updating apt package lists

```bash
apt update
```

Kubernetes requires an underlying container runtime. There are [multiple options](https://kubernetes.io/docs/setup/production-environment/container-runtimes/) but I have decided to go with containerd for simplicity. To install containerd, you need to perform the following steps.

```bash
# Add prerequisites
apt install -y apt-transport-https ca-certificates curl gpg

# Add Docker's official GPG key:
install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
chmod a+r /etc/apt/keyrings/docker.asc

# Add the docker repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] " \
  "https://download.docker.com/linux/debian " \
  "$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  tee /etc/apt/sources.list.d/docker.list > /dev/null
apt update
apt install -y containerd.io

# configure containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
systemctl restart containerd
```

Now we need to install `kubeadm`, `kubelet`, and `kubectl`. 

`kubeadm` is used for bootstrapping new clusters and adding nodes to an existing cluster, `kubelet` is the actual daemon that runs on the server to run containers, and `kubectl` is used to interact with a running kube cluster.

```bash
# add kubernetes keyring and apt sources
curl -fsSLo /etc/apt/keyrings/kubernetes.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | \
  tee /etc/apt/sources.list.d/kubernetes.list

# install packages
apt update
apt install -y kubelet kubeadm kubectl

# kubernetes should always be manually updated, so we tell apt not to update these packages any time others are updated
apt-mark hold kubelet kubeadm kubectl

# enable the kubelet
systemctl enable --now kubelet
```

Now, kubernetes requires a bit more system-level configuration before it can be brought online as a node in a cluster. First off, kubernetes does not support running on systems that have swap enabled. There is an [in depth discussion](https://github.com/kubernetes/kubernetes/issues/53533) that explains why Kubernetes will not run with a system that is swap enabled. Feel free to read about it yourself, its outside of scope for this article. 

```bash
# disable all swap immediately
swapoff -a
```

It is important that you also remove any swap that is configured to in  `/etc/fstab`. I recommend editing the file manually and commenting out any lines that reference swap partitions for safety.

Kube also needs a couple kernel modules to be enabled and sysctls configured to work properly, so lets make that happen too.

```bash
# load the required modules into the currently running kernel
modprobe br_netfilter overlay

# configure the modules to load at boot
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

# define the sysctls needed for kube to work correctly
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# activate the above sysctls in the running system
sysctl -p /etc/sysctl.d/k8s.conf
```

This next step is semi-optional, but required to install the kube dashboard and a bunch of other stuff as well, so definitely handy to have.

```bash
# install helm
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | tee /usr/share/keyrings/helm.gpg > /dev/null
echo "deb [arch=$(dpkg --print-architecture) " \
  "signed-by=/usr/share/keyrings/helm.gpg] " \
  "https://baltocdn.com/helm/stable/debian/ all main" | \
  tee /etc/apt/sources.list.d/helm-stable-debian.list
apt update
apt install helm
```

You can technically skip this step, but I still like to manually pull the kube images before spinning up the node.

```bash
# pull the kube images (this isn't strictly required, but )
kubeadm config images pull
```

## Prepare OPNsense

Now, this is where things start to diverage a little from the typical bare-metal kube deploy, and something I had to figure out on my own. It isn't very difficult, but hopefully documenting this step will help someone else (or myself in the future, most likely). In order to have multiple control planes, which is definitely what I want in my own lab, you must provide a load-balanced DNS address for the kubernetes API. I've read that you can do a simple roud-robin DNS, but I want the cluster to be as robust as possible. I decided to go with HA Proxy provided by OPNsense.

```note
NOTE:

OPNsense requires that the system be running the latest version in order to install plugins, so be prepared to update the router and reboot the box.
```

- Login to OPNsense router
- Navigate to `System > Firmware > Status`, click `Check for Updates`.
  [![OPNsense sreenshot 1](opnsense_1.png?width=900px)](opnsense_1.png)
- Follow the directions for updating the system
- After updating, log back into the system
- Navigate to `System > Firmware > Plugins`
- Scroll down and find `os-haproxy` and click the `+` sign to install it
- Refresh the page in the browser and navigate to `Services > HAProxy > Settings`
- Click on the `Real Servers` and click the `+` at the bottom of the list
  [![OPNsense sreenshot 2](opnsense_2.png?width=900px)](opnsense_2.png)
- Configure the first server you are going to use to bootstrap
  [![OPNsense sreenshot 3](opnsense_3.png?width=900px)](opnsense_3.png)
  - Enter the name
  - Set `type` to `static`
  - Add the IP or FQDN of the server
  - Set port to `6443`
  - Click `Save`
- Click on the `Virtual Services` tab drop down, then click `Backend Pools` and click on the `+` in the list
- Configure the backend pool
  [![OPNsense sreenshot 4](opnsense_4.png?width=900px)](opnsense_4.png)
  - For `Name`, enter `kube-cp-backend` (or any name you want)
  - Optionally set `Description` to `kubernetes control planes` (or something similiar, its only for display)
  - Set `Mode` to `TCP (Layer 4)`
  - Add your server under `Servers`
  - Check `Enable Health Checking` (this may not be necessary, but its how I have mine configured and is working)
  - Click `Save`
- Click on the `Virtual Services` tab drop down, then click `Public Services` and click on the `+` in the list
  [![OPNsense sreenshot 5](opnsense_5.png?width=900px)](opnsense_5.png)
  - For `Name`, enter `kube-cp-frontend` or whatever name you want
  - Enter the address and port on which you want the load balancer to listen on the OPNsense router in the form of `<IP>:<PORT>`, for example `192.168.1.1:6443`, assuming your router LAN ip address is `192.168.1.1`
  - Set `Type` to `TCP`
  - For `Default Backend Pool`, select the backend server pool you configured above
  - Under `Stickiness table`, set `Table type` to `only store IPv4 addresses`
  - Click `Save`
- Click on the `Settings` tab, then click `Service`
- Check the box for `Enable HAProxy`
- Click `Save & Test Syntax`
- Assuming the syntax is OK, click `Apply`
  

The final step is technically optional and will possibly be configured outside of OPNsense and is completely dependent on your LAN services. You should configure an internal DNS entry that points to the LAN ip address of your router, something dedicated for Kubernetes. This shouldn't be technically required, however, because you can provide `kubeadm` with the internal IP address of your router below instead of a DNS name.

## Bootstrap the new cluster

This step is only required on one node because it is what creates the actual Kubernetes cluster. Subsequent nodes will be added to the cluster and require slightly different steps.

More to follow...