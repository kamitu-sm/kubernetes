# Installing Kubernetes Cluster on Centos 8 (kubeadm) #

We will be creating a single control-plane cluster using kubeadm. We will also be providing for future expansion into a multi control plane cluster with a * ***stacked etcd store***. This is good for a small setups that will be scaled out in future. 

***Note (What is meant by stacked etcd store)***
*In the introduction a note was made that the etcd store and the k8s cluster are considered seperate. There are two options for highly available kubernetes clusters.*

*1. With stacked control plane nodes. The etcd members and control plane nodes are co-located.* 

*2. With an external etcd cluster, the control plane nodes and etcd members are separated.* 

*The reason this setup cannot scale to an external etcd is because we are not setting up a etcd store separately prior to initialising the k8s cluster. One is not able to separate the etcd store after intialising the cluster. but the steps remain as below with the difference being how the intitial master node is intialised specifying the etcd store in a yaml config file.*


## Pre-flight checklist ## 

1. Two or more machines running CentOS 8.
2. 2 GiB or more of RAM per machine, any less leaves little room for your apps.
3. At least 2 CPUs on the machine that you use as a control-plane node.
4. IP plan for the setup

Node Role  | IP Address
---------- | -------------
Master     | 192.168.100.29
Worker     | 192.168.100.29

5. Full network connectivity among all machines in the cluster.
6. A version of kubeadm that can deploy the version of Kubernetes that you want to use in your new cluster.
7. Choose a Pod network add-on, and verify whether it requires any arguments to be passed to kubeadm when initialising the cluster. Depending on which third-party provider you choose, you might need to set the --pod-network-cidr to a provider-specific value. ***We will be using Calico***


## Preparing all servers ##

There are a few things to be done to get the servers ready. You need to perform the following task on all servers. 


1. Disable the Selinux

The containers need to access the host file system; therefore SELinux needs to be disabled. Edit the /etc/selinux/config file and change SELINUX=enforcing to SELINUX=disabled or permissive

```bash
 # sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

After this a reboot is required


2. Disable firewalld


As long as firewalld(the system firewall manager) is enabled, DNS resolution inside docker containers does not work. We are going to be using iptables, disable firewalld daemon

```bash
# systemctl disable firewalld
# systemctl stop firewalld
```

Allow all traffic between the cluster nodes by creating an accept all rule on iptables

```bash
# iptables -P FORWARD ACCEPT
```

3. Disable swap and enable port forwarding

To ensure that packets are properly processed by IP tables during filtering and port forwarding, set the net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file. For the containers to cluster to work properly disable swap, or alternatively set vm.swappiness=0. 
Modify the kernel parameters for filtering and port forwarding. Below we are creating a kernel parameter file k8s.conf and placing it in the /etc/sysctl.d directory. This will make the changes persistent between reboots. Additionally we are also setting vm.swappiness to 0, meaning to disable swap.

```bash
# vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
```

Force systclt daemon to read the contents of its configuration files

```bash
# sysctl --system
```

4. Set the DNS names
If you will not be using a DNS server, edit /etc/hosts file to contain the following:

```bash
# cat /etc/hosts
192.168.100.29 k8s_master_1
192.168.100.29 k8s_api_lb
192.168.100.40 k8s_node_2
```

5. Install and configure Docker

```bash
# yum config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
# yum install docker-ce
Last metadata expiration check: 0:02:40 ago on Wed 04 Dec 2019 07:58:38 PM EAT.
Error:
Problem: package docker-ce-3:19.03.5-3.el7.x86_64 requires containerd.io >= 1.2.2-3,
but none of the providers can be installed
- cannot install the best candidate for the job
- package containerd.io-1.2.10-3.2.el7.x86_64 is excluded
- package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
- package containerd.io-1.2.2-3.el7.x86_64 is excluded
- package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
- package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
- package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not
only best candidate packages)
```

Installation of containerd.io > 1.2.0-3.el7 is blocked, which is a dependency of docker-ce. We will follow the one of the recommendations given above and install the no best community edition of docker

```bash
# yum install docker-ce --nobest
```

Setup daemon to use systemd instead of cgroupsfs (Requirements for kubeadm, the same will be done on the kubelet via initial config file). We dont want a scenario were we have the daemon being managed by both systemd and cgroupfs for stability reasons


```bash
# cat > /etc/docker/daemon.json <<EOF
{
"exec-opts": ["native.cgroupdriver=systemd"],
"log-driver": "json-file",
"log-opts": {
"max-size": "100m"
},
"storage-driver": "overlay2",
"storage-opts": [
"overlay2.override_kernel_check=true"
]
}
EOF
```

The make the directory for the docker systemd service

```bash
# mkdir -p /etc/systemd/system/docker.service.d
```

Restart Docker


```bash
# systemctl daemon-reload
# systemctl enable docker
# systemctl restart docker
```

6. Install kubeadm, kubectl and kubelet


Kubernetes packages are not available in the default CentOS 8 repositories. Make sure to create a kuberentes repo file with the contents below before you do the install.

```bash
# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
# yum repolist
# yum install kubelet kubeadm kubectl
# systemctl daemon-reload
# systemctl enable kubelet
# systemctl restart kubelet
```
