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
