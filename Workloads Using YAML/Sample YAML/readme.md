# Introduction #
This folder contains sample object yaml files for different objects identified with the name of the yaml file, also below is a helpful introduction on various k8s objects commonly used in the clsuter

## Kubernetes replication ##
Typically you would want to replicate your containers (and thereby your applications) for several reasons, including:
1. Reliability: By having multiple versions of an application, you prevent problems if one or more fails.
2. Load balancing: Having multiple versions of a container enables you to easily send traffic to different instances to prevent overloading of a single instance or node. 
3. Scaling: When load does become too much for the number of existing instances, Kubernetes enables you to easily scale up your application, adding additional instances as needed.

### So how do we do replication? ###
There are three different forms of replication: 
* Replication Controller
* Replica Sets
* Deployments.

***Replication Controller***
The Replication Controller is the original form of replication in Kubernetes.  It’s being replaced by Replica Sets, but it’s still in wide use, so it’s worth understanding what it is and how it works. A Replication Controller is a structure that enables you to easily create multiple pods, then make sure that that number of pods always exists. If a pod does crash, the Replication Controller replaces it. Replication Controllers also provide other benefits, such as the ability to scale the number of pods, and to update or delete multiple pods with a single command.

***Replica Sets***
Replica Sets are a sort of hybrid, in that they are in some ways more powerful than Replication Controllers, and in others they are less powerful. Replica Sets are declared in essentially the same way as Replication Controllers, except that they have more options for the selector.

***Deployments***
Deployments are intended to replace Replication Controllers.  They provide the same replication functions (through Replica Sets) and also the ability to rollout changes and roll them back if necessary.

## What is a Service in Kubernetes? ##
A Service enables network access to a set of Pods in Kubernetes.

Services select Pods based on their labels. When a network request is made to the service, it selects all Pods in the cluster matching the service's selector, chooses one of them, and forwards the network request to it.

<img src="https://github.com/kamitu-sm/kubernetes/blob/master/Workloads%20Using%20YAML/Sample%20YAML/service.png" alt="KUBERNETES SERVICE OVERVIEW">

The type property in the Service's spec determines how the service is exposed to the network. It changes where a Service is able to be accessed from. The possible types are ClusterIP, NodePort, and LoadBalancer

1. ClusterIP – The default value. The service is only accessible from within the Kubernetes cluster – you can’t make requests to your Pods from outside the cluster!
2. NodePort – This makes the service accessible on a static port on each Node in the cluster. This means that the service can handle requests that originate from outside the cluster.
3. LoadBalancer – The service becomes accessible externally through a cloud provider's load balancer functionality. GCP, AWS, Azure, and OpenStack offer this functionality. The cloud provider will create a load balancer, which then automatically routes requests to your Kubernetes Service
