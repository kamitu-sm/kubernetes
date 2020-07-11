# Installing Kubernetes Cluster on Centos 8 (kubeadm) #

We will be creating a single control-plane cluster using kubeadm. We will also be providing for future expansion into a multi control plane cluster with a stacked etcd store. This is good for a small setups that will be scaled out in future.

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


