# Introduction #

This is a guide for installing a single control-plane Kubernetes cluster on Centos8 using kubeadm. We will also be providing for future expansion into a multi control plane cluster with a ***stacked etcd store***. This is good for a small setups that will be scaled out in future. 

***Note (What is meant by stacked etcd store)***
*In the Kubernetes architecture introduction a note was made that the etcd store and the k8s cluster are considered seperate. There are two options for highly available kubernetes clusters.*

*1. With stacked control plane nodes. The etcd members and control plane nodes are co-located.* 

*2. With an external etcd cluster, the control plane nodes and etcd members are separated.* 

*The reason this setup cannot scale to an external etcd is because we are not setting up a etcd store separately prior to initialising the k8s cluster, but the steps remain as below with the difference being how the intitial master node is intialised specifying the etcd store in a yaml config file.*

## Table of file contents ##
### 1. Pre-flight checklist ### 
Preparing your servers for the cluster setup
### 2. Initializing your Cluster ###
Creating your cluster

