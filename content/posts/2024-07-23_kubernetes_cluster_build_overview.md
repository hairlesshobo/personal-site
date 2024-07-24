+++
title = 'Bare metal Kubernetes Cluster: Overview'
date = "2024-07-23"
draft = false
tags = ['home lab']
keywords = ['Kubernetes Cluster Build']
summary = 'The overview of my plan to build a bare-metel Kubernetes cluster in the home lab'
+++

## Status

This project is still a work-in-progress, but I intend to revise this article as I go along. 

## Overview

I am in the process of building a self-hosted, bare metal Kubernetes cluster in my lab. The purpose of this article is to show how I built it and what I found that did work and those things that did not work. I have no doubt that not every use case will be covered here. This will likely end up being a multi-part article as well, given the size of this project.

## Goals

I currently use `docker-compose` for all of my container orchestration locally. While this is better than running individual containers using `docker run` with a bunch of shell scripts, it is still very limited. I have considered running Docker Swarm but considering the number of companies using Kubernetes compared to Swarm, it seemed like the best idea to go with Kube. I primarily want Kubernetes for the distributed nature of it. Overall requirements:

* Multiple systems (ideally 3 or more), all of which are control planes for redundancy
* Automatic node selection for apps that require extra hardware such as GPU or a specific connected USB device
* Distributed, persistent storage for stateful data
* Automatic routing and local network DNS configuration
* Access to CIFS network file shares

## Plan

* Deploy a full instance of Kubernetes using [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/)
* Persistent, distributed storage provided by [Ceph](https://docs.ceph.com/en/reef/start/) so that pods can easily migrate to other nodes without manual intervention
* Use [external-dns](https://github.com/kubernetes-sigs/external-dns) with the powerdns plugin to enable automatic DNS updates. 
* Use [Calico](https://docs.tigera.io/calico/latest/about/) as the CNI for Kubernetes so that I can easily setup BGP routing from my existing OPNsense routers to easily expose internal kubernetes IP addresses to my LAN
* Use [csi-driver-smb](https://github.com/kubernetes-csi/csi-driver-smb) to allow mounting of CIFS shares to pods

