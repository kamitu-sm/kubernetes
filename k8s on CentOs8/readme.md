# Installing Kubernetes Cluster on Centos 8 (kubeadm) #

We will be creating a single control-plane cluster using kubeadm. We will also be providing for future expansion into a multi control plane cluster with a stacked etcd store. This is good for a small setups that will be scaled out in future. 

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


