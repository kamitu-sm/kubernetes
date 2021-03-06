# Pre-flight checklist # 

## Planning Requirements ##
1. Two or more machines running CentOS 8.
2. 2 GiB or more of RAM per machine, any less leaves little room for your apps.
3. At least 2 CPUs on the machine that you use as a control-plane node.
4. IP plan for the setup. Turning a single control plane cluster created without --control-plane-endpoint into a highly available cluster is not supported by kubeadm. We are using this arguement to point to the same master node using a DNS entry **k8s_api_lb**, Later you can modify this DNS entry to point to the address of your load-balancer in an high availability scenario without having to intialise your cluster.

Node Role        | IP Address       | DNS Name
---------------- | -----------------| ------------
Master           | 192.168.100.29   | k8s-master-1
Worker           | 192.168.100.34   | k8s-node-2
API Loadbalancer | 192.168.100.29   | k8s-api-lb (DNS Entry pointing back to master) 


5. Full network connectivity among all machines in the cluster.
6. A version of kubeadm that can deploy the version of Kubernetes that you want to use in your new cluster.
7. Choose a Pod network add-on, and verify whether it requires any arguments to be passed to kubeadm when initialising the cluster. Depending on which third-party provider you choose, you might need to set the --pod-network-cidr to a provider-specific value. ***We will be using Calico***


## Preparing all servers ##

There are a few things to be done to get the servers ready. You need to perform the following task on all servers. 


1. **Disable the Selinux**

The containers need to access the host file system; therefore SELinux needs to be disabled. Edit the /etc/selinux/config file and change SELINUX=enforcing to SELINUX=disabled or permissive

```bash
 # sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
```

After this a reboot is required


2. **Disable firewalld**


As long as firewalld(the system firewall manager) is enabled, DNS resolution inside docker containers does not work. We are going to be using iptables, disable firewalld daemon

```bash
# systemctl disable firewalld
# systemctl stop firewalld
```

Allow all traffic between the cluster nodes by creating an accept all rule on iptables

```bash
# iptables -P FORWARD ACCEPT
```

3. **Disable swap and enable port forwarding**

For runtime type
```bash
# swapoff -a
```

To make this configuration persistent between reboots, edit the /etc/fstab file and comment out all lines for mounting any swap file system into the system

```bash
# sed -i 's/^.*swap/#&/' /etc/fstab
```

To ensure that packets are properly processed by IP tables during filtering and port forwarding, set the net.bridge.bridge-nf-call-iptables to ‘1’ in your sysctl config file. For the containers in the cluster to work properly disable swap, also set vm.swappiness=0. 
Below we are creating a kernel parameter file k8s.conf and placing it in the /etc/sysctl.d directory. This file will modify the kernel parameters for filtering, port forwarding and vm.swappiness. This will make the changes persistent between reboots.

```bash
# vi /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
```

Force sysctl daemon to read the contents of its configuration files

```bash
# sysctl --system
```

4. **Set the DNS names**

If you will not be using a DNS server, edit /etc/hosts file to contain the following:

```bash
# cat /etc/hosts
192.168.100.29 k8s-master-1
192.168.100.29 k8s-api-lb
192.168.100.34 k8s-node-2
```

5. **Install Docker**

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

Installation of containerd.io-1.2.10-3.2.el7.x86_64 is filtered out by modular filtering, which is a dependency of docker-ce. We will follow the one of the recommendations given above and install the no best community edition of docker

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

6. **Configure Docker daemon to use systemd**

Setup the daemon to use systemd instead of cgroupsfs (Requirements for kubeadm, the same will be done on the kubelet via initial config file).
When systemd is chosen as the init system for a linux distribution.  The init process generates and consumes a root cgroup and acts as a cgroup manager.  Systemd has a tight integration with cgroups and will allocate cgroups per process. While it's possible to configure docker and kubelet to use cgroupfs this means that there will then be two different cgroup managers. At the end of the day, cgroups are used to allocate and constrain resources that are allocated to processes. A single cgroup manager will simplify the view of what resources are being allocated and will by default have a more consistent view of the resources available / in use. When we have two managers we end up with two views of those available resources. We have seen cases in the field where nodes that are configured to use cgroupfs for kubelet and docker and systemd for the rest can become unstable under resource pressure. Changing the settings such that docker and kubelet use systemd as a cgroup-driver stabilized the systems. ***Advise from the k8s community***

This file */etc/docker/daemon.json* might need to be created


```bash
# vi /etc/docker/daemon.json
# cat /etc/docker/daemon.json 
{
	"exec-opts": ["native.cgroupdriver=systemd"],
	"log-driver": "json-file",
	"log-opts": {
		"max-size": "100m"
	},
	"storage-driver": "overlay2",
	"storage-opts": ["overlay2.override_kernel_check=true"]
}

```

Restart Docker

```bash
# systemctl daemon-reload
# systemctl enable docker
# systemctl restart docker
```

7. **Install kubeadm, kubectl and kubelet**


Kubernetes packages are not available in the default CentOS 8 repositories. Make sure to create a kuberenetes repo file with the contents below before you do the install.

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
#
# yum repolist
repo id                                                            repo name
AppStream                                                          CentOS-8 - AppStream
BaseOS                                                             CentOS-8 - Base
docker-ce-stable                                                   Docker CE Stable - x86_64
extras                                                             CentOS-8 - Extras
kubernetes                                                         Kubernetes
#
# yum install kubelet kubeadm kubectl
Kubernetes                                                                                                    154  B/s | 454  B     00:02    
Kubernetes                                                                                                    2.0 kB/s | 1.8 kB     00:00    
Importing GPG key 0xA7317B0F:
 Userid     : "Google Cloud Packages Automatic Signing Key <gc-team@google.com>"
 Fingerprint: D0BC 747F D8CA F711 7500 D6FA 3746 C208 A731 7B0F
 From       : https://packages.cloud.google.com/yum/doc/yum-key.gpg
Is this ok [y/N]: y
Importing GPG key 0xBA07F4FB:
 Userid     : "Google Cloud Packages Automatic Signing Key <gc-team@google.com>"
 Fingerprint: 54A6 47F9 048D 5688 D7DA 2ABE 6A03 0B21 BA07 F4FB
 From       : https://packages.cloud.google.com/yum/doc/yum-key.gpg
Is this ok [y/N]: y
Kubernetes                                                                                                    3.0 kB/s | 975  B     00:00    
Importing GPG key 0x3E1BA8D5:
 Userid     : "Google Cloud Packages RPM Signing Key <gc-team@google.com>"
 Fingerprint: 3749 E1BA 95A8 6CE0 5454 6ED2 F09C 394C 3E1B A8D5
 From       : https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
Is this ok [y/N]: y
Kubernetes                                                                                                     22 kB/s |  98 kB     00:04    
Dependencies resolved.
==============================================================================================================================================
 Package                                    Architecture               Version                           Repository                      Size
==============================================================================================================================================
Installing:
 kubeadm                                    x86_64                     1.18.5-0                          kubernetes                     8.8 M
 kubectl                                    x86_64                     1.18.5-0                          kubernetes                     9.5 M
 kubelet                                    x86_64                     1.18.5-0                          kubernetes                      21 M
Installing dependencies:
 conntrack-tools                            x86_64                     1.4.4-10.el8                      BaseOS                         204 k
 cri-tools                                  x86_64                     1.13.0-0                          kubernetes                     5.1 M
 kubernetes-cni                             x86_64                     0.8.6-0                           kubernetes                      18 M
 libnetfilter_cthelper                      x86_64                     1.0.0-15.el8                      BaseOS                          24 k
 libnetfilter_cttimeout                     x86_64                     1.0.0-11.el8                      BaseOS                          24 k
 libnetfilter_queue                         x86_64                     1.0.2-11.el8                      BaseOS                          30 k
 socat                                      x86_64                     1.7.3.3-2.el8                     AppStream                      302 k

Transaction Summary
==============================================================================================================================================
Install  10 Packages

Total download size: 62 M
Installed size: 265 M

..................................................
```

Finally enable and restart the kubelet service

```bash
# systemctl daemon-reload
# systemctl enable kubelet
# systemctl restart kubelet
```
