## Kubernetes YAML Basics ##

### Understanding Kubernetes objects ###
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:
* What containerized applications are running (and on which nodes)
* The resources available to those applications
* The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

### Kubernetes YAML file ###
All Kuberenetes YAML files have the map structure below with the specific key values depending on the type of object being configured

```yaml
---
 apiVersion:
 kind:
 metadata:
 spec:
```

* ***apiVersion*** - Which version of the Kubernetes API you're using to create this object
* ***kind*** - What kind of object you want to create
* ***metadata*** - Data that helps uniquely identify the object, including a name string, UID, and optional namespace
* ***spec*** - What state you desire for the object

### Relationship between apiVersion and Kind ###
An object definition in Kubernetes requires an apiVersion field. When Kubernetes has released an update that is avaialable for you to use or changes something in its API, a new apiVersion is created.

However, the official Kubernetes documentation provides little guidance on apiVersion. This guide gives you a cheat sheet on which version to use, explains each version, and gives you the timeline of releases.

### API groups ###
To make it easier to extend its API, Kubernetes implements API groups. The API group is specified in a REST path and in the apiVersion field of a serialized object.

There are several API groups in a cluster:
* The core group, also referred to as the legacy group. This is at the REST path /api/v1 and uses apiVersion: v1.
* The named groups, which are at the REST path /apis/$GROUP_NAME/$VERSION, and use apiVersion: $GROUP_NAME/$VERSION (e.g. apiVersion: batch/v1). The Kubernetes API reference has a full list of available API groups.

### API versioning ###
To make it easier to eliminate fields or restructure resource representations, Kubernetes supports multiple API versions, for each group

1. ***alpha***

API versions with ‘alpha’ in their name are early candidates for new functionality coming into Kubernetes. These may contain bugs and are not guaranteed to work in the future.

2. ***beta***

‘beta’ in the API version name means that testing has progressed past alpha level, and that the feature will eventually be included in Kubernetes. Although the way it works might change, and the way objects are defined may change completely, the feature itself is highly likely to make it into Kubernetes in some form.

3. ***stable***

These do not contain ‘alpha’ or ‘beta’ in their name. They are safe to use.

### Which apiVersion should I use? ###

***v1***
This was the first stable release of the Kubernetes API. It contains many core objects.

***apps/v1***
apps is the most common API group in Kubernetes, with many core objects being drawn from it and v1. It includes functionality related to running applications on Kubernetes, like Deployments, RollingUpdates, and ReplicaSets.

***autoscaling/v1***
This API version allows pods to be autoscaled based on different resource usage metrics. This stable version includes support for only CPU scaling, but future alpha and beta versions will allow you to scale based on memory usage and custom metrics.

***batch/v1***
The batch API group contains objects related to batch processing and job-like tasks (rather than application-like tasks like running a webserver indefinitely). This apiVersion is the first stable release of these API objects.

***batch/v1beta1***
A beta release of new functionality for batch objects in Kubernetes, notably including CronJobs that let you run Jobs at a specific time or periodicity.

***certificates.k8s.io/v1beta1***
This API release adds functionality to validate network certificates for secure communication in your cluster. You can read more on the official docs.

***extensions/v1beta1***
This version of the API includes many new, commonly used features of Kubernetes. Deployments, DaemonSets, ReplicaSets, and Ingresses all received significant changes in this release.

Note that in Kubernetes 1.6, some of these objects were relocated from extensions to specific API groups (e.g. apps). When these objects move out of beta, expect them to be in a specific API group like apps/v1. Using extensions/v1beta1 is becoming deprecated—try to use the specific API group where possible, depending on your Kubernetes cluster version.

***policy/v1beta1***
This apiVersion adds the ability to set a pod disruption budget and new rules around pod security.

***rbac.authorization.k8s.io/v1***
This apiVersion includes extra functionality for Kubernetes role-based access control. This helps you to secure your cluster. Check out the official blog post.


Kind  (Object)             |   	apiVersion
---------------------------|------------------------------------
CertificateSigningRequest  |	certificates.k8s.io/v1beta1
ClusterRoleBinding         |	rbac.authorization.k8s.io/v1
ClusterRole                |	rbac.authorization.k8s.io/v1
ComponentStatus            |	v1
ConfigMap                  |	v1
ControllerRevision         |	apps/v1
CronJob                    |	batch/v1beta1
DaemonSet                  |	extensions/v1beta1
Deployment                 |	extensions/v1beta1
Endpoints                  |	v1
Event                      |	v1
HorizontalPodAutoscaler    |	autoscaling/v1
Ingress                    |	extensions/v1beta1
Job                        |	batch/v1
LimitRange                 |	v1
Namespace                  |	v1
NetworkPolicy              |	extensions/v1beta1
Node                       |	v1
PersistentVolumeClaim      |	v1
PersistentVolume           |	v1
PodDisruptionBudget        |	policy/v1beta1
Pod                        |	v1
PodSecurityPolicy          |	extensions/v1beta1
PodTemplate                |	v1
ReplicaSet                 |	extensions/v1beta1
ReplicationController      |	v1
ResourceQuota              |	v1
RoleBinding                |	rbac.authorization.k8s.io/v1
Role                       |	rbac.authorization.k8s.io/v1
Secret                     |	v1
ServiceAccount             |	v1
Service                    |	v1
StatefulSet                |	apps/v1
