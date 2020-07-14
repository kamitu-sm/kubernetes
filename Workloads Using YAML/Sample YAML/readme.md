# Introduction #
This folder contains sample object yaml files for different objects identified with the name of the yaml file, also below is a helpful introduction on various k8s objects commonly used in the clsuter

## Replication in Kubernetes? ##
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

## Service in Kubernetes ##
A Service enables network access to a set of Pods in Kubernetes.

Services select Pods based on their labels. When a network request is made to the service, it selects all Pods in the cluster matching the service's selector, chooses one of them, and forwards the network request to it.

<img src="https://github.com/kamitu-sm/kubernetes/blob/master/Workloads%20Using%20YAML/Sample%20YAML/service.png" alt="KUBERNETES SERVICE OVERVIEW">

The type property in the Service's spec determines how the service is exposed to the network. It changes where a Service is able to be accessed from. The possible types are ClusterIP, NodePort, and LoadBalancer

1. ClusterIP – The default value. The service is only accessible from within the Kubernetes cluster – you can’t make requests to your Pods from outside the cluster!
2. NodePort – This makes the service accessible on a static port on each Node in the cluster. This means that the service can handle requests that originate from outside the cluster.
3. LoadBalancer – The service becomes accessible externally through a cloud provider's load balancer functionality. GCP, AWS, Azure, and OpenStack offer this functionality. The cloud provider will create a load balancer, which then automatically routes requests to your Kubernetes Service

## DaemonSet in Kubernetes ##
DaemonSets are used to ensure that some or all of your K8S nodes run a copy of a pod, which allows you to run a daemon on every node.

When you add a new node to the cluster, a pod gets added to match the nodes. Similarly, when you remove a node from your cluster, the pod is put into the trash. Deleting a DaemonSet cleans up the pods that it previously created.

### Why use DaemonSets? ###
Now that we understand DaemonSets, here are some examples of why and how to use it:

1. To run a daemon for cluster storage on each node, such as:
* glusterd
* ceph
2. To run a daemon for logs collection on each node, such as:
* fluentd
* logstash
3. To run a daemon for node monitoring on every node, such as:
* Prometheus Node Exporter
* collectd
* Datadog agent

## ConfigMap in Kubernetes ##
A ConfigMap stores configuration settings that your Kubernetes Pods consume.

### How does a ConfigMap work? ###
A ConfigMap is a dictionary of key-value pairs that store configuration settings for your applications. 

First, create a ConfigMap in your cluster by tweaking our sample YAML to your needs.

```yaml
kind: ConfigMap 
apiVersion: v1 
metadata:
  name: example-configmap 
data:
  # Configuration values can be set as key-value properties
  database: mongodb
  database_uri: http://localhost:27017
  
  # Or set as complete file contents (even JSON!)
  keys: | 
    image.public.key=771 
    rsa.public.key=42
```
Second, consume to ConfigMap in your Pods and use its values. Set envFrom to a reference to the ConfigMap you’ve created.

```yaml
kind: Pod 
apiVersion: v1 
metadata:
  name: pod-env-var 
spec:
  containers:
    - name: env-var-configmap
      image: nginx:1.7.9 
      envFrom:
        - configMapRef:
            name: example-configmap
```

confirm inside your container

```bash
$ kubectl exec -it pod-env-var sh
# env
DATABASE=mongodb
DATABASE_URI=http://localhost:27017
image.public.key=771
rsa.public.key=42
```
