+++
title = 'Bare metal Kubernetes Cluster: Bootstrap [WIP]'
date = "2024-07-24"
draft = false
tags = ['home lab']
keywords = ['Kubernetes Cluster Build']
summary = 'Steps I followed to bootstrap my new Kubernetes cluster'
+++

## Process (rough draft)

This article will outline the process I followed to bootstrap a brand new Kubernetes cluster from scratch, and subsequently adding additional nodes to the cluster.

As my systems are all running vanilla Debian, the package manager commands will all be using `apt`. 

```warning
WARNING:

I am bad and use root for work like this instead of using sudo while working on personal systems, so you will need to `su -` to root before starting. For the record, I know this is bad practice and I only do this on personal servers. You should 100% use sudo on the commands below where appropriate instead.
```

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


More to follow...