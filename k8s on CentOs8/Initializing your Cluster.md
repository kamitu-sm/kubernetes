# Initializing your Cluster #

Please make sure you have read through the pre-flight checklist and prepared the setup before attempting the below 

### Considerations about --apiserver-advertise-address and --control-plane-endpoint ###

While --apiserver-advertise-address can be used to set the advertise address for this particular control-plane node's API server, --control-plane-endpoint can be used to set the shared endpoint for all control-plane nodes. --control-plane-endpoint allows both IP addresses and DNS names that can map to IP addresses. The --control-plane-endpoint flag should be set to the DNS and port of the load balancer(for high availability scenario).

Here is an example mapping:

192.168.0.102 cluster-endpoint
Where 192.168.0.102 is the IP address of this node and cluster-endpoint is a custom DNS name that maps to this IP. This will allow you to pass --control-plane-endpoint=cluster-endpoint to kubeadm init and pass the same DNS name to kubeadm join. Later you can modify cluster-endpoint to point to the address of your load-balancer in an high availability scenario.

Turning a single control plane cluster created without --control-plane-endpoint into a highly available cluster is not supported by kubeadm

### POD Network ###
You can install only one Pod network per cluster. There are couple of options for this including calico, weave and flannel. Some POD networks have certain restrictions on the IP address range for the pods. Calico will automatically detect which IP address range to use for pod IPs based on the value provided via the --pod-network-cidr flag or via kubeadm's configuration.

### Initialization Config File ###
We will be using a YAML file for kubeadm intialization. This needs only be done once on one of the Master nodes. Create a YAML file with the contents below in your current working directory. 

```bash
# 
#
# cat kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- token: "y40psg.1gl8f0exzc11pedz"
  description: "My bootstrap token"
  ttl: "24h"
  usages:
  - authentication
  - signing
  groups:
  - system:bootstrappers:kubeadm:default-node-token
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.100.29
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
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
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controlPlaneEndpoint: "k8s-api-lb:6443"
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
controlPlaneEndpoint: "k8s-api-lb:6443"
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

## Step 1: ***Initializing the Control Plane*** ##

The control-plane node is the machine where the control plane components run, including etcd (the cluster database) and the API Server (which the kubectl command line tool communicates with).

On the Master run the command below

```bash
# sudo kubeadm config images pull
W0711 17:58:31.015389   53936 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.18.5
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.18.5
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.18.5
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.18.5
[config/images] Pulled k8s.gcr.io/pause:3.2
[config/images] Pulled k8s.gcr.io/etcd:3.4.3-0
[config/images] Pulled k8s.gcr.io/coredns:1.6.7
#
#
#
# sudo kubeadm init --config kubeadm-config.yaml --upload-certs 2>&1 |tee kubeadm_init_ouput.txt 
W0711 18:52:25.675159   57351 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.5
[preflight] Running pre-flight checks
	[WARNING FileExisting-tc]: tc not found in system path
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master-1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local k8s-api-lb] and IPs [10.96.0.1 192.168.100.29]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master-1 localhost] and IPs [192.168.100.29 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-master-1 localhost] and IPs [192.168.100.29 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
W0711 18:52:32.176657   57351 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0711 18:52:32.186125   57351 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0711 18:52:32.187267   57351 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 20.502719 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
0bbe9c06c89dde9a9829269abe502d884edb53c00644a4107800477547726b74
[mark-control-plane] Marking the node k8s-master-1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master-1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: y40psg.1gl8f0exzc11pedz
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
    --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 \
    --control-plane --certificate-key 0bbe9c06c89dde9a9829269abe502d884edb53c00644a4107800477547726b74

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
    --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 
    
#
#
# mv kubeadm_init_ouput.txt kubeadm_init_ouput_11_07_2020_18_52.bak
```

1. The first command pre pulls the required images for the adminstrative tasks
2. The second command initialises the control plane, we are piping using tee to a file because of the interesting output this command generates including the ***kubeadm join*** command for future nodes. The --upload-certs flag is used to upload the certificates that should be shared across all the control-plane instances to the cluster. If instead, you prefer to copy certs across control-plane nodes manually or using automation tools, please remove this flag. There are a lot of advantages for using this
3. The last command backs up the output incase you run the command in future and need to compare outputs



## Step 2: ***Follow the instructions from the above output*** ##

Run the commands below as a normal user to be able to use kubectl

```bash

  $  mkdir -p $HOME/.kube
  $  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  $  sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
```
***Make sure you are running the kubectl command as the user in this step***

## Step 3: ***Initializing the POD Network*** ##

When we run the command ***kubectl get pods --all-namespaces***, we notice that the coredns pod is in pending status

```bash
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-bv4bj               0/1     Pending   0          11m
kube-system   coredns-66bff467f8-vsqr4               0/1     Pending   0          11m
kube-system   etcd-k8s-master-1                      1/1     Running   0          11m
kube-system   kube-apiserver-k8s-master-1            1/1     Running   0          11m
kube-system   kube-controller-manager-k8s-master-1   1/1     Running   0          11m
kube-system   kube-proxy-vtrbm                       1/1     Running   0          11m
kube-system   kube-scheduler-k8s-master-1            1/1     Running   0          11m
```

On the Master run the command ***kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml***. 

```bash
$ kubectl apply -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
$ 
$ 
```

Once a Pod network has been installed, you can confirm that it is working by checking that the CoreDNS Pod is Running in the output of ***kubectl get pods --all-namespaces***. And once the CoreDNS Pod is up and running, you can continue by joining your nodes.

```bash
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS     RESTARTS   AGE
kube-system   calico-kube-controllers-76d4774d89-jmsfh   0/1     Pending    0          78s
kube-system   calico-node-n79kk                          0/1     Init:0/3   0          78s
kube-system   coredns-66bff467f8-bv4bj                   0/1     Pending    0          13m
kube-system   coredns-66bff467f8-vsqr4                   0/1     Pending    0          13m
kube-system   etcd-k8s-master-1                          1/1     Running    0          13m
kube-system   kube-apiserver-k8s-master-1                1/1     Running    0          13m
kube-system   kube-controller-manager-k8s-master-1       1/1     Running    0          13m
kube-system   kube-proxy-vtrbm                           1/1     Running    0          13m
kube-system   kube-scheduler-k8s-master-1                1/1     Running    0          13m
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-76d4774d89-jmsfh   0/1     ContainerCreating   0          3m27s
kube-system   calico-node-n79kk                          1/1     Running             0          3m27s
kube-system   coredns-66bff467f8-bv4bj                   1/1     Running             0          15m
kube-system   coredns-66bff467f8-vsqr4                   1/1     Running             0          15m
kube-system   etcd-k8s-master-1                          1/1     Running             0          15m
kube-system   kube-apiserver-k8s-master-1                1/1     Running             0          15m
kube-system   kube-controller-manager-k8s-master-1       1/1     Running             0          15m
kube-system   kube-proxy-vtrbm                           1/1     Running             0          15m
kube-system   kube-scheduler-k8s-master-1                1/1     Running             0          15m
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-76d4774d89-jmsfh   1/1     Running   0          4m23s
kube-system   calico-node-n79kk                          1/1     Running   0          4m23s
kube-system   coredns-66bff467f8-bv4bj                   1/1     Running   0          16m
kube-system   coredns-66bff467f8-vsqr4                   1/1     Running   0          16m
kube-system   etcd-k8s-master-1                          1/1     Running   0          16m
kube-system   kube-apiserver-k8s-master-1                1/1     Running   0          16m
kube-system   kube-controller-manager-k8s-master-1       1/1     Running   0          16m
kube-system   kube-proxy-vtrbm                           1/1     Running   0          16m
kube-system   kube-scheduler-k8s-master-1                1/1     Running   0          16m
```
