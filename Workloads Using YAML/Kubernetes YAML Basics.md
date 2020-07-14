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
* The core group, also referred to as the legacy group, is at the REST path /api/v1 and uses apiVersion: v1.
* Named groups are at REST path /apis/$GROUP_NAME/$VERSION, and use apiVersion: $GROUP_NAME/$VERSION (e.g. apiVersion: batch/v1). The Kubernetes API reference has a full list of available API groups.

### API versioning ###
To make it easier to eliminate fields or restructure resource representations, Kubernetes supports multiple API versions, for each group

***alpha***
API versions with ‘alpha’ in their name are early candidates for new functionality coming into Kubernetes. These may contain bugs and are not guaranteed to work in the future.

***beta***
‘beta’ in the API version name means that testing has progressed past alpha level, and that the feature will eventually be included in Kubernetes. Although the way it works might change, and the way objects are defined may change completely, the feature itself is highly likely to make it into Kubernetes in some form.

***stable***
These do not contain ‘alpha’ or ‘beta’ in their name. They are safe to use.

### Which apiVersion should I use? ###
