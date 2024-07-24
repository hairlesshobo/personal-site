+++
title = 'Home Lab'
date = "2024-07-22"
lastmod = "2024-07-24"
tags = ['Home Lab']
draft = false
summary = 'Home lab overview'
+++

My current home lab consists of 6 Linux servers, dual redundant OPNsense routers, 8 VLANS, an enterprise grade 10g fiber network, and countless containerized local services. I also operate my own email server, which is not nearly as simple to do as it used to be 15-20 years ago, but it is still a fun challenge! 

I have a RTX 3060 12GB in one of my Linux servers (starfox) that I use for ML acceleration for a variety of purposes including security, family history, and just general ML fun. 

Pictures below show the current state of the lab. This is without a doubt the cleanest lab I have ever had setup. There is still a ton I want to do with it, and lots of ethernet and fiber drops I need to put in the house, but as of this writing, its summer in southeast Georgia and I don't want to stroke out in the attic from heat.

[![Lab pic 1 - 2024-07-19](2024-07-19_home-lab-1.jpg?width=400px)](2024-07-19_home-lab-1.jpg)
[![Lab pic 2 - 2024-07-19](2024-07-19_home-lab-2.jpg?width=400px)](2024-07-19_home-lab-2.jpg)

## Hardware and System List

### Network

- TP-Link 8-Port 10GE SFP+ Managed Switch [[TL-SX3008F](https://www.tp-link.com/us/business-networking/managed-switch/tl-sx3008f/)]
- MikroTik 24-Port 1G + 2-Port 10GE SFP+ Managed Switch [[CSS326-24G-2S+](https://mikrotik.com/product/CSS326-24G-2SplusRM)]
- TP-Link 16-Port 1G PoE + 2-Port 1G SFP Managed Switch [[TL-SG1218MPE](https://www.tp-link.com/us/business-networking/easy-smart-switch/tl-sg1218mpe/)]

### Systems

* router1: Primary router
  * OPNsense
  * Custom build: AMD A8-3850 / 8G DDR3 
  * 2u rackmount case
  * Intel X550-T2 dual 10G RJ45 

* router2: Failover router
  * OPNsense
  * Lenovo m93p: Intel i7-4765T / 16GB DDR3
  * USFF machine on 1u rack shelf

* starfox: Primary server
  * Debian
  * Custom build: AMD AMD Ryzen 7 5700G / 64G DDR4
  * 4u rackmount case [link](https://www.rosewill.com/rosewill-rsv-l4412u-black/p/9SIA072GJ92847)
  * Nvidia RTX 3060 12G [link](https://www.pny.com/geforce-rtx-3060-12gb-xlr8-gaming-revel-epic-x-rgb-df)
  * Intel X520-SR2 dual 10G SFP+ 
  * 6x 6TB WD Red Plus w/ OpenZFS

* donkey: Secondary server
  * Debian
  * Custom build: AMD Ryzen 5 3600 / 16GB DDR4
  * ancient 4u rack case (labeled AudioLog by Mercom - given to me ~20 years ago)
  * Intel X520-SR2 dual 10G SFP+

* mule: Secondary storage server
  * Debian
  * HP ProLiant ML110 G7 / Intel Xeon E3-1220 / 8GB DDR3 ECC
  * Desktop server form factor, on rack shelf
  * 4x 3TB WD Red Plus w/ OpenZFS

* kube-cp1: experiment server
  * Debian
  * OptiPlex 3040 / Intel i3-6100T / 4GB DDR3
  * USFF machine on 1u rack shelf

* kube-n1: experiment server
  * Debian
  * Lenovo m93p: Intel i5-4570T / 16GB DDR3
  * USFF machine on 1u rack shelf

* kube-n1: experiment server
  * Debian
  * Lenovo m93p: Intel i7-4765T / 16GB DDR3
  * USFF machine on 1u rack shelf


## Network services / Daemons

* AgentDVR
* Apache reverse proxy
* Apache WebDAV server
* CloudLog - Amateur Radio logbook server
* CodeProject AI server
* Crashplan
* CumulusClips
* Elasticsearch + Kibana
* Email server stack (self-hosted externally): Dovecot + Postfix + SpamAssassain + Sieve + Roundcube
* FFSync server
* Filestash
* Gitea
* HomeAssistant (HassOS hosted using KVM on donkey)
* Joplin note server
* MariaDB
* Minio object storage server
* Nextcloud personal cloud server
* Nginx
* Ollama LLM server
* Photoprism
* PostgreSQL
* PowerDNS auth server + dnsdist + PowerDNS Admin (2x)
* Restic backup
* Samba file server
* Samba Windows AD server
* TinyLLM chat bot
* Woodpecker CI/CD runner

## ML Models

* Bringing Old Photos Back To Life [[link](https://github.com/microsoft/Bringing-Old-Photos-Back-to-Life)]
* Deoldify [[link](https://github.com/jantic/DeOldify)]
* Face Processing [[link](https://github.com/codeproject/CodeProject.AI-FaceProcessing)]
* llama 3 [[link](https://ollama.com/library/llama3)]
* Super resolution [[link](https://github.com/codeproject/CodeProject.AI-SuperResolution)]
* Whisper [[link](https://github.com/openai/whisper)]
* YOLO v8 object detection [[link](https://github.com/codeproject/CodeProject.AI-ObjectDetectionYOLOv8)]

## Previous Tech

This is a non-exhaustive list of things I have previously ran in my lab but do not currently use.

* Operating Systems
  * FreeNAS
  * Mandrake Linux
  * pfSense
  * Slackware
  * TrueNAS
  * Windows Server 2008/2012/2016
  * Zeroshell
* Services
  * Bacula
  * Duplicati
  * Microsoft IIS
  * Microsoft SQL Server
  * Starwind iSCSI Server