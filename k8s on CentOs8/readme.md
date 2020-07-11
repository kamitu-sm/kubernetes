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
4. IP plan for the setup. Turning a single control plane cluster created without --control-plane-endpoint into a highly available cluster is not supported by kubeadm. We are using this arguement to point to the same master node using a DNS entry **k8s_api_lb**, Later you can modify this DNS entry to point to the address of your load-balancer in an high availability scenario without having to intialise your cluster.

Node Role        | IP Address       | DNS Name
---------------- | -----------------| ------------
Master           | 192.168.100.29   | k8s_master_1
Worker           | 192.168.100.40   | k8s_node_2
API Loadbalancer | 192.168.100.29   | k8s_api_lb (DNS Entry pointing back to master) 


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

5. Install Docker

```bash
# yum config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
Adding repo from: https://download.docker.com/linux/centos/docker-ce.repo
# 
# 
# yum repolist -v
Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, needs-restarting, playground, repoclosure, repodiff, repograph, repomanage, reposync
YUM version: 4.2.17
cachedir: /var/cache/dnf
User-Agent: constructed: 'libdnf (CentOS Linux 8; generic; Linux.x86_64)'
repo: using cache for: AppStream
AppStream: using metadata from Wed 08 Jul 2020 02:25:16 EAT.
repo: using cache for: BaseOS
BaseOS: using metadata from Wed 08 Jul 2020 02:25:12 EAT.
repo: using cache for: extras
extras: using metadata from Fri 05 Jun 2020 03:15:26 EAT.
repo: using cache for: docker-ce-stable
docker-ce-stable: using metadata from Tue 23 Jun 2020 01:31:42 EAT.
Last metadata expiration check: 0:00:43 ago on Sat 11 Jul 2020 16:18:36 EAT.
Completion plugin: Generating completion cache...

Repo-id            : AppStream
Repo-name          : CentOS-8 - AppStream
Repo-revision      : 8.2.2004
Repo-distro-tags      : [cpe:/o:centos:centos:8]:  , 8, C, O, S, e, n, t
Repo-updated       : Wed 08 Jul 2020 02:25:16 EAT
Repo-pkgs          : 5,359
Repo-available-pkgs: 4,933
Repo-size          : 6.2 G
Repo-mirrors       : http://mirrorlist.centos.org/?release=8&arch=x86_64&repo=AppStream&infra=stock
Repo-baseurl       : http://centos.mirror.liquidtelecom.com/8.2.2004/AppStream/x86_64/os/ (9 more)
Repo-expire        : 172,800 second(s) (last: Sat 11 Jul 2020 16:18:36 EAT)
Repo-filename      : /etc/yum.repos.d/CentOS-AppStream.repo

Repo-id            : BaseOS
Repo-name          : CentOS-8 - Base
Repo-revision      : 8.2.2004
Repo-distro-tags      : [cpe:/o:centos:centos:8]:  , 8, C, O, S, e, n, t
Repo-updated       : Wed 08 Jul 2020 02:25:12 EAT
Repo-pkgs          : 1,675
Repo-available-pkgs: 1,673
Repo-size          : 1.0 G
Repo-mirrors       : http://mirrorlist.centos.org/?release=8&arch=x86_64&repo=BaseOS&infra=stock
Repo-baseurl       : http://centos.mirror.liquidtelecom.com/8.2.2004/BaseOS/x86_64/os/ (9 more)
Repo-expire        : 172,800 second(s) (last: Sat 11 Jul 2020 16:08:00 EAT)
Repo-filename      : /etc/yum.repos.d/CentOS-Base.repo

Repo-id            : docker-ce-stable
Repo-name          : Docker CE Stable - x86_64
Repo-revision      : 1592865102
Repo-updated       : Tue 23 Jun 2020 01:31:42 EAT
Repo-pkgs          : 79
Repo-available-pkgs: 71
Repo-size          : 1.9 G
Repo-baseurl       : https://download.docker.com/linux/centos/7/x86_64/stable
Repo-expire        : 172,800 second(s) (last: Sat 11 Jul 2020 16:08:01 EAT)
Repo-filename      : /etc/yum.repos.d/docker-ce.repo

Repo-id            : extras
Repo-name          : CentOS-8 - Extras
Repo-revision      : 1591316131
Repo-updated       : Fri 05 Jun 2020 03:15:26 EAT
Repo-pkgs          : 20
Repo-available-pkgs: 20
Repo-size          : 236 k
Repo-mirrors       : http://mirrorlist.centos.org/?release=8&arch=x86_64&repo=extras&infra=stock
Repo-baseurl       : http://centos.mirror.liquidtelecom.com/8.2.2004/extras/x86_64/os/ (9 more)
Repo-expire        : 172,800 second(s) (last: Sat 11 Jul 2020 16:16:00 EAT)
Repo-filename      : /etc/yum.repos.d/CentOS-Extras.repo
Total packages: 7,133
#
#
#
# yum install docker-ce
Last metadata expiration check: 0:00:10 ago on Sat 11 Jul 2020 16:18:36 EAT.
Error: 
 Problem: package docker-ce-3:19.03.12-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.4-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.5-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.6-3.3.el7.x86_64 is filtered out by modular filtering
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

Installation of containerd.io > 1.2.0-3.el7 is blocked, which is a dependency of docker-ce. We will follow the one of the recommendations given above and install the no best community edition of docker

```bash
# yum install docker-ce --nobest
Last metadata expiration check: 0:03:40 ago on Sat 11 Jul 2020 16:18:36 EAT.
Dependencies resolved.

 Problem: package docker-ce-3:19.03.12-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.4-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.5-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.6-3.3.el7.x86_64 is filtered out by modular filtering
==============================================================================================================================================
 Package                                Architecture     Version                                             Repository                  Size
==============================================================================================================================================
Installing:
 docker-ce                              x86_64           3:18.09.1-3.el7                                     docker-ce-stable            19 M
Installing dependencies:
 checkpolicy                            x86_64           2.9-1.el8                                           BaseOS                     348 k
 container-selinux                      noarch           2:2.124.0-1.module_el8.2.0+305+5e198a41             AppStream                   47 k
 containerd.io                          x86_64           1.2.0-3.el7                                         docker-ce-stable            22 M
 docker-ce-cli                          x86_64           1:19.03.12-3.el7                                    docker-ce-stable            38 M
 libcgroup                              x86_64           0.41-19.el8                                         BaseOS                      70 k
 policycoreutils-python-utils           noarch           2.9-9.el8                                           BaseOS                     251 k
 python3-audit                          x86_64           3.0-0.17.20191104git1c2f876.el8                     BaseOS                      86 k
 python3-libsemanage                    x86_64           2.9-2.el8                                           BaseOS                     127 k
 python3-policycoreutils                noarch           2.9-9.el8                                           BaseOS                     2.2 M
 python3-setools                        x86_64           4.2.2-2.el8                                         BaseOS                     601 k
Enabling module streams:
 container-tools                                         rhel8                                                                               
Skipping packages with broken dependencies:
 docker-ce                              x86_64           3:19.03.12-3.el7                                    docker-ce-stable            24 M

Transaction Summary
==============================================================================================================================================
Install  11 Packages
Skip      1 Package

Total download size: 83 M
Installed size: 341 M
Is this ok [y/N]: 

```

6. Configure Docker daemon
Setup the daemon to use systemd instead of cgroupsfs (Requirements for kubeadm, the same will be done on the kubelet via initial config file). We don't want a scenario were we have the daemon being managed by both systemd and cgroupfs for stability reasons


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

7. Install kubeadm, kubectl and kubelet


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
