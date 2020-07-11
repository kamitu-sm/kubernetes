Initializing your control-plane node 

The control-plane node is the machine where the control plane components run, including etcd (the cluster database) and the API Server (which the kubectl command line tool communicates with).

Considerations about apiserver-advertise-address and ControlPlaneEndpoint 

While --apiserver-advertise-address can be used to set the advertise address for this particular control-plane node's API server, --control-plane-endpoint can be used to set the shared endpoint for all control-plane nodes. --control-plane-endpoint allows both IP addresses and DNS names that can map to IP addresses. The --control-plane-endpoint flag should be set to the DNS and port of the load balancer(for high availability scenario).

Here is an example mapping:

192.168.0.102 cluster-endpoint
Where 192.168.0.102 is the IP address of this node and cluster-endpoint is a custom DNS name that maps to this IP. This will allow you to pass --control-plane-endpoint=cluster-endpoint to kubeadm init and pass the same DNS name to kubeadm join. Later you can modify cluster-endpoint to point to the address of your load-balancer in an high availability scenario.

Turning a single control plane cluster created without --control-plane-endpoint into a highly available cluster is not supported by kubeadm


The YAML config file

```
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

The --upload-certs flag is used to upload the certificates that should be shared across all the control-plane instances to the cluster. If instead, you prefer to copy certs across control-plane nodes manually or using automation tools, please remove this flag and refer to Manual certificate distribution section below.
