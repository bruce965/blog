---
title: Oracle Cloud Free ARM Kubernetes Cluster
date: 2023-08-05 15:01:34 +0200
categories: [Software, Kubernetes]
tags: [Kubernetes, Docker, cloud, Oracle, free, Linux]
---

This is a guide to install a single-node Kubernetes cluster on a free
Oracle Cloud ARM server.


## Account

Create an Oracle Cloud account from here:
https://cloud.oracle.com/


## Server

Create a new compute instance
https://cloud.oracle.com/compute/instances/create

Name: oracle-k8s (any valid name is fine)

Create in compartment: your-username (root)

Placement:
leave everything as default.

Security:
leave everything as default.

Image and shape:
- Canonical Ubuntu 22.04
- VM.Standard.A1.Flex with 4 OCPUS and 24 GB of RAM

Networking:
- Primary network: Create new virtual cloud network
  - New virtual cloud network name: vcn-k8s
  - Create in compartment: your-username (root)
- Subnet: Create a new public subnet
  - New subnet name: subnet-k8s
  - Create in compartment: your-username (root)
  - CIDR block: 10.0.0.0/24
- Public IPv4 address: Assign a public IPv4 address

Add SSH keys:
- Generate a key pair for me
  - Save private key

Boot volume:
- Specify a custom boot volume size: yes
  - Boot volume size: 200 GB (you may use 100 GB and leave 100 GB for the free x86_64 servers)
  - Boot volume performance: 10 VPU
- Use in-transit encryption: yes
- Encrypt this volume with a key that you manage: no

> Oracle might give you an estimate cost for the boot volume which you can disregard,
> the free tier includes 200 GB of block volume storage.

Live migration: active

Click "Create" to create the machine.


## Connecting

At the end of previous step, you should have been redirected to the instance page,
which you can open by clicking on your instance's name from the instances page:
https://cloud.oracle.com/compute/instances/

You can use [PuTTY](https://putty.org/) to open a SSH conection to your server.

Use the "Public IP address" to connect, and log-in as the "Username".

SSH port 22 is already exposed.


<!--
## Firewall

By default, the firewall is configured to drop certain packets, it should be disabled.

```bash
# Save current rules
sudo iptables-save > ~/iptables_rules.bak

# Delete DROP and REJECT rules
cat ~/iptables_rules.bak | grep -v DROP | grep -v REJECT | sudo iptables-restore

# Apply changes
sudo netfilter-persistent save
sudo systemctl restart iptables
```

Ports must also be opened on VCN subnets, accessible from here:
https://cloud.oracle.com/networking/vcns/

- Click on your VCN's name (vcn-k8s)
- From the subnets section, click on your subnet's name (subnet-k8s)
- From the security lists section, click on "Default Security List for vcn-k8s"
- Click "Add Ingress Rules" again
  - Stateless: no
  - Source Type: CIDR
  - Source CIDR: 10.0.0.0/24
  - IP Protocol: TCP
  - Source Port Range: (leave empty to allow all ports)
  - Destination Port Range: 6443
  - Click "Add Ingress Rule"
< !--
- Click "Add Ingress Rules" again
  - Stateless: no
  - Source Type: CIDR
  - Source CIDR: 0.0.0.0/0
  - IP Protocol: All Protocols
  - Click "Add Ingress Rule"
-- >
-->


## Installing Kubernetes

The first requirement to install Kubernetes is Docker:
https://docs.docker.com/engine/install/ubuntu/

```bash
sudo install --mode=0755 --directory /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install --yes docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

To validate Docker's installation, run this command and ensure there are no errors:

```bash
sudo docker run hello-world
```

The second requirement is cri-dockerd:
https://github.com/Mirantis/cri-dockerd/releases

Unfortunately this tool doesn't come with pre-built packages for ARM,
so it will have to be built from source.

```bash
# Install build tools (marking as "auto" will make the installation temporary)
sudo apt install --yes make golang-go
sudo apt-mark auto make golang-go

# Clone Git repository
cd ~
git clone https://github.com/Mirantis/cri-dockerd.git
cd cri-dockerd

# Build (this will take a while and won't report any progress)
make cri-dockerd

# Install
sudo mkdir -p /usr/local/bin
sudo install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
sudo install packaging/systemd/* /etc/systemd/system
sudo sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
sudo systemctl daemon-reload
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
```

It's now time to install kubeadm:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```bash
sudo apt install --yes apt-transport-https

curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

echo \
  "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \
  sudo tee /etc/apt/sources.list.d/kubernetes.list
  
sudo apt update
sudo apt install --yes kubelet kubeadm kubectl

# Disable Kubernetes auto-update
sudo apt-mark hold kubelet kubeadm kubectl
```


## Initializing a cluster

At last, a Kubernetes cluster can be initialized:
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

> The control plane endpoint cannot be easily changed after initialization.
> If you plan to add more nodes to this cluster, you should replace `127.0.0.1`
> with the private IP of your server, which you can check from your instance page.
> Or even better, create a domain name and set it to the private IP of your server,
> then use it as the control plane endpoint, so you can change it in the future.
{: .prompt-info }

```bash
sudo kubeadm init --control-plane-endpoint=127.0.0.1 --cri-socket=unix:///var/run/cri-dockerd.sock
```

<!--
Create a _kubeadm-config.yaml_ template file with
`kubeadm config print init-defaults > ~/kubeadm-config.yaml`.

You can edit this file with `nano ~/kubeadm-config.yaml`.

Make sure to set the `criSocket` to `unix:///var/run/cri-dockerd.sock` to use
Docker instead of containerd.

And install it with `kubeadm init --config ~/kubeadm-config.yaml`.
-->

