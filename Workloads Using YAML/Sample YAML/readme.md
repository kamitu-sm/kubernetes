## Introduction ##
This folder contains sample object yaml files for different objects identified with the name of the yaml file, also below is a helpful introduction on various k8s objects commonly used in the clsuter

### Kubernetes replication ###
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
