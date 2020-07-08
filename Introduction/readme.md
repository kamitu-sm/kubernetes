Understanding Kubernetes 
==============================

1. ***Introduction to Kubernetes*** 
Container orchestration is the automatic process of managing or scheduling the work of individual containers for applications within multiple clusters. Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services (i.e. it’s a container orchestration Engine), that facilitates both declarative configuration and automation

2. ***Kubernetes architecture and terminology*** 
By understanding the architecture and terminologies we are able to better understand the various configurations that are needed to build and maintain your kubernetes cluster.
Let us start with a couple of terminologies that are required to understand the architecture and then introduce further terminologies as we explain the architecture.
        1. Container:
Put simply, a container consists of an entire runtime environment: an application, plus all its dependencies, libraries and other binaries, and configuration files needed to run it, bundled into one package. By containerizing the application platform and its dependencies, differences in OS distributions and underlying infrastructure are abstracted away.
 Containers are a solution to the problem of how to get software to run reliably when moved from one computing environment to another. This could be from a developer's laptop to a test environment, from a staging environment into production, and perhaps from a physical machine in a data center to a virtual machine in a private or public cloud. Problems arise when the supporting software environment is not identical. "You're going to test using Python 2.7, and then it's going to run on Python 3 in production and something weird will happen. Or you'll rely on the behavior of a certain version of an SSL library and another one will be installed. You'll run your tests on Debian and production is on Red Hat and all sorts of weird things happen."
        2. Pod:
In kubernetes world a pod is the base unit of the kubernetes cluster, all configurations are done in reference to pods. A pod is equivalent to a VM in the virtualization world.  It’s the scheduling unit of kubernetes. A Pod is a group of one or more containers (such as Docker containers), with shared storage/network, and a specification for how to run the containers. A Pod's contents are always co-located and co-scheduled, and run in a shared context A Pod models an application-specific "logical host" - it contains one or more application containers which are relatively tightly coupled.
Containers within a Pod share an IP address and port space, and can find each other via localhost. Containers in different Pods have distinct IP addresses and cannot communicate by IPC without special configuration. These containers usually communicate with each other via Pod IP addresses.

        3. Node
 A node is a physical server or virtual machines with an operating systems running on top of it.
The kubernetes cluster is made up of two classifications of nodes
1.	Master nodes (Primary responsibility is management of the cluster components)
2.	Worker node (Doing the actual workload also known as minions)
At minimum you need one master node and one or many worker nodes. In a high availability cluster you can have multiple master nodes. 

3. ***Basic Components of the Kubernetes Architecture***  


![picture alt](https://github.com/kamitu-sm/kubernetes/blob/master/Introduction/k8s-basic-architecture.png "KUBERNETES ARCHITECTURE") 


