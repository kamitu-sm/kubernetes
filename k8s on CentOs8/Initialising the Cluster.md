# Initializing your control-plane node #

The control-plane node is the machine where the control plane components run, including etcd (the cluster database) and the API Server (which the kubectl command line tool communicates with).

### Considerations about apiserver-advertise-address and ControlPlaneEndpoint ###

While --apiserver-advertise-address can be used to set the advertise address for this particular control-plane node's API server, --control-plane-endpoint can be used to set the shared endpoint for all control-plane nodes. --control-plane-endpoint allows both IP addresses and DNS names that can map to IP addresses. The --control-plane-endpoint flag should be set to the DNS and port of the load balancer(for high availability scenario).

Here is an example mapping:

192.168.0.102 cluster-endpoint
Where 192.168.0.102 is the IP address of this node and cluster-endpoint is a custom DNS name that maps to this IP. This will allow you to pass --control-plane-endpoint=cluster-endpoint to kubeadm init and pass the same DNS name to kubeadm join. Later you can modify cluster-endpoint to point to the address of your load-balancer in an high availability scenario.

Turning a single control plane cluster created without --control-plane-endpoint into a highly available cluster is not supported by kubeadm

### POD Network ###
You can install only one Pod network per cluster. Calico will automatically detect which IP address range to use for pod IPs based on the value provided via the --pod-network-cidr flag or via kubeadm's configuration.

### Kubeadm Initialization Config File ###
We will be using a YAML file for kubeadm intialisation. This needs only be done once on one of the Master nodes. Create a YAML file with the contents below in your current working directory. 

```bash
# 
# kubeadm token generate
y40psg.1gl8f0exzc11pedz
# vi kubeadm-config.yaml
# cat kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
- system:bootstrappers:kubeadm:default-node-token
token: y40psg.1gl8f0exzc11pedz
ttl: 24h0m0s
usages:
- signing
- authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.100.29
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master-1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
kubeletExtraArgs:
  cgroup-driver: systemd
---
apiServer:
  extraArgs:
    authorization-mode: Node,RBAC
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta1
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "k8s_api_lb:6443"
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
kind: ClusterConfiguration
kubernetesVersion: stable
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}
```

***Notes on the YAML file***

***token***

```bash
token: y40psg.1gl8f0exzc11pedz
```

The token in the YAML file is obtained from running the command ***kubeadm token generate***. You can generate your offhead as long as it conforms to the set standards.

```bash
# 
# kubeadm token generate
y40psg.1gl8f0exzc11pedz
```

***--apiserver-advertise-address***

```bash
localAPIEndpoint:
  advertiseAddress: 192.168.100.29
  bindPort: 6443
```
This is equivalent to the --apiserver-advertise-address flag as discussed above 

***--control-plane-endpoint***

```bash
controlPlaneEndpoint: "k8s_api_lb:6443"
```
This is equivalent to the --control-plane-endpoint flag as discussed above

***etcd***
```bash
etcd:
  local:
    dataDir: /var/lib/etcd
```

This is the config for a stacked etcd store. For a external store this configuration changes to 

```bash
etcd:
    external:
        endpoints:
        - https://ETCD_0_IP:2379
        - https://ETCD_1_IP:2379
        - https://ETCD_2_IP:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
```
***POD Networking***
```bash
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"
```
  
This is equivalent to the --pod-network-cidr flag. Some POD networks have certain restrictions on this range. Calico will automatically detect which IP address range to use for pod IPs based on the value provided via the --pod-network-cidr flag or via kubeadm's configuration. Makes sure it does not confilct with your node IP addressing

## STEPS ##

On the Master run the command below

sudo kubeadm init --config kubeadm-config.yaml --upload-certs

The --upload-certs flag is used to upload the certificates that should be shared across all the control-plane instances to the cluster. If instead, you prefer to copy certs across control-plane nodes manually or using automation tools, please remove this flag and refer to Manual certificate distribution section below.
