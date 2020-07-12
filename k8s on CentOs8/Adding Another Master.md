
# Adding another master (optional) #

This is included to show you how easy it is to scale the control plane of the setup. To make this setup a proper high availability a proxy server/ Load balancer should be set up with your control plane nodes behind it. This load balancer distributes traffic to all healthy control plane nodes in its target list. The health check for an apiserver is a TCP check on the port the kube-apiserver listens on (default value :6443). The load balancer must be able to communicate with all control plane nodes on the apiserver port. It must also allow incoming traffic on its listening port.

DNS entry for **k8s-api-lb** should point to this load balancer. The listening port for the loadbalancer for this setup should be **6443**.

## Let's get to it ##

For each additional control plane node you should execute the join command that was previously given to you by the kubeadm init output on the first node. It should look something like this:

```
You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
    --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 \
    --control-plane --certificate-key 0bbe9c06c89dde9a9829269abe502d884edb53c00644a4107800477547726b74
```
The --control-plane flag tells kubeadm join to create a new control plane.


The --certificate-key will cause the control plane certificates to be downloaded from the kubeadm-certs Secret in the cluster and be decrypted using the given key.

So we are following this, go into the node you are trying to join as root and paste this command

```bash
[root@k8s-node-2 ~]#   kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
>     --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 \
>     --control-plane --certificate-key 0bbe9c06c89dde9a9829269abe502d884edb53c00644a4107800477547726b74
[preflight] Running pre-flight checks
	[WARNING FileExisting-tc]: tc not found in system path
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks before initializing the new control plane instance
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[download-certs] Downloading the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
error execution phase control-plane-prepare/download-certs: error downloading certs: error downloading the secret: Secret "kubeadm-certs" was not found in the "kube-system" Namespace. This Secret might have expired. Please, run `kubeadm init phase upload-certs --upload-certs` on a control plane to generate a new one
To see the stack trace of this error execute with --v=5 or higher

```

Seems that we took longer than two hours to join our node, in the init output there is a warning like this

```
Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.
```

so we go back to our first master and do as instructed

```
[root@k8s-master-1 ~]# kubeadm init phase upload-certs --upload-certs
W0712 11:38:37.935774   80178 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[upload-certs] Storing the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[upload-certs] Using certificate key:
7c5a2e54fb22e91dc71f0a0c61e9d9a7933082f5073d7937e45692297546144f
```
Now we have a new certifcation key, which we will be substituting into the previous command in the --certificate-key option

```bash
[root@k8s-node-2 ~]# kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
>      --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 \
>      --control-plane --certificate-key 7c5a2e54fb22e91dc71f0a0c61e9d9a7933082f5073d7937e45692297546144f
[preflight] Running pre-flight checks
	[WARNING FileExisting-tc]: tc not found in system path
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[preflight] Running pre-flight checks before initializing the new control plane instance
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[download-certs] Downloading the certificates in Secret "kubeadm-certs" in the "kube-system" Namespace
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-node-2 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local k8s-api-lb] and IPs [10.96.0.1 192.168.100.34]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-node-2 localhost] and IPs [192.168.100.34 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [k8s-node-2 localhost] and IPs [192.168.100.34 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[certs] Using the existing "sa" key
[kubeconfig] Generating kubeconfig files
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
W0712 11:45:25.251664    3231 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0712 11:45:25.259473    3231 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0712 11:45:25.260519    3231 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[check-etcd] Checking that the etcd cluster is healthy
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[etcd] Announced new etcd member joining to the existing etcd cluster
[etcd] Creating static Pod manifest for "etcd"
[etcd] Waiting for the new etcd member to join the cluster. This can take up to 40s
{"level":"warn","ts":"2020-07-12T11:45:41.399+0300","caller":"clientv3/retry_interceptor.go:61","msg":"retrying of unary invoker failed","target":"passthrough:///https://192.168.100.34:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = context deadline exceeded"}
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[mark-control-plane] Marking the node k8s-node-2 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-node-2 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]

This node has joined the cluster and a new control plane instance was created:

* Certificate signing request was sent to apiserver and approval was received.
* The Kubelet was informed of the new secure connection details.
* Control plane (master) label and taint were applied to the new node.
* The Kubernetes control plane instances scaled up.
* A new etcd member was added to the local/stacked etcd cluster.

To start administering your cluster from this node, you need to run the following as a regular user:

	mkdir -p $HOME/.kube
	sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	sudo chown $(id -u):$(id -g) $HOME/.kube/config

Run 'kubectl get nodes' to see this node join the cluster.
```

Confirm your work by running the commands below on a node that is already configured to administer the cluster

``` bash
[stephen@k8s-master-1 ~]$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master-1   Ready    master   16h     v1.18.5
k8s-node-2     Ready    master   3m32s   v1.18.5
```
Note that the cluster now has two masters

``` bash
[stephen@k8s-master-1 ~]$ kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-76d4774d89-jmsfh   1/1     Running   1          16h   10.244.196.6     k8s-master-1   <none>           <none>
kube-system   calico-node-n79kk                          1/1     Running   1          16h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   calico-node-vrw5w                          1/1     Running   0          12m   192.168.100.34   k8s-node-2     <none>           <none>
kube-system   coredns-66bff467f8-bv4bj                   1/1     Running   1          17h   10.244.196.4     k8s-master-1   <none>           <none>
kube-system   coredns-66bff467f8-vsqr4                   1/1     Running   1          17h   10.244.196.5     k8s-master-1   <none>           <none>
kube-system   etcd-k8s-master-1                          1/1     Running   1          17h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   etcd-k8s-node-2                            1/1     Running   0          12m   192.168.100.34   k8s-node-2     <none>           <none>
kube-system   kube-apiserver-k8s-master-1                1/1     Running   1          17h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-apiserver-k8s-node-2                  1/1     Running   0          12m   192.168.100.34   k8s-node-2     <none>           <none>
kube-system   kube-controller-manager-k8s-master-1       1/1     Running   3          17h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-controller-manager-k8s-node-2         1/1     Running   2          12m   192.168.100.34   k8s-node-2     <none>           <none>
kube-system   kube-proxy-vtrbm                           1/1     Running   1          17h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-proxy-zjdx4                           1/1     Running   0          12m   192.168.100.34   k8s-node-2     <none>           <none>
kube-system   kube-scheduler-k8s-master-1                1/1     Running   3          17h   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-scheduler-k8s-node-2                  1/1     Running   2          12m   192.168.100.34   k8s-node-2     <none>           <none>

```
Notice the admin pods like the etcd, kube-apiserver, kube-controller-manager, kube-proxy and kube-scheduler are now on both nodes.

This proves that the cluster can support high availability. I will be removing this node as a way of showing you how to go about it and also since it was not part of the setup.

### How to delete a master node ###

On any other master node with kubectl configured

1. ***Find the node***

``` bash
[stephen@k8s-master-1 ~]$ kubectl get nodes
NAME           STATUS   ROLES    AGE     VERSION
k8s-master-1   Ready    master   16h     v1.18.5
k8s-node-2     Ready    master   3m32s   v1.18.5
```

2. ***Drain it*** 

This is just incase it was carrying other pods

``` bash
[stephen@k8s-master-1 ~]$ kubectl drain k8s-node-2 --ignore-daemonsets
node/k8s-node-2 cordoned
WARNING: ignoring DaemonSet-managed Pods: kube-system/calico-node-kfcq7, kube-system/kube-proxy-gpqlq
node/k8s-node-2 drained
[stephen@k8s-master-1 ~]$ 

``` 

3. ***Delete it***

``` bash
[stephen@k8s-master-1 ~]$ kubectl delete node k8s-node-2
node "k8s-node-2" deleted
[stephen@k8s-master-1 ~]$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master-1   Ready    master   17h   v1.18.5
``` 
wait until there are no longer any pods on the deleted master, give it like 10 minutes

``` bash
[stephen@k8s-master-1 ~]$ kubectl get nodes
NAME           STATUS   ROLES    AGE   VERSION
k8s-master-1   Ready    master   19m   v1.18.5
[stephen@k8s-master-1 ~]$ kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE   IP               NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-76d4774d89-nclh2   1/1     Running   0          17m   10.244.196.2     k8s-master-1   <none>           <none>
kube-system   calico-node-xb7cg                          1/1     Running   0          17m   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   coredns-66bff467f8-b559w                   1/1     Running   0          19m   10.244.196.3     k8s-master-1   <none>           <none>
kube-system   coredns-66bff467f8-pnhkg                   1/1     Running   0          19m   10.244.196.1     k8s-master-1   <none>           <none>
kube-system   etcd-k8s-master-1                          1/1     Running   0          19m   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-apiserver-k8s-master-1                1/1     Running   0          19m   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-controller-manager-k8s-master-1       1/1     Running   1          19m   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-proxy-rkhst                           1/1     Running   0          19m   192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-scheduler-k8s-master-1                1/1     Running   1          19m   192.168.100.29   k8s-master-1   <none>           <none>
``` 

4. ***(Advise against this)On the node to be removed. Run kubeadm reset***


- ***I have noticed that this command crushes the entire cluster. From the output below its clear that the command tries to connect to the api server. I suspect its a bug and will raise a bug report.***
- ***Maybe you should not do this***


``` bash
[root@k8s-node-2 ~]# kubeadm reset
[reset] Reading configuration from the cluster...
[reset] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
W0712 13:36:05.221585   18147 reset.go:99] [reset] Unable to fetch the kubeadm-config ConfigMap from cluster: failed to get config map: Get https://k8s-api-lb:6443/api/v1/namespaces/kube-system/configmaps/kubeadm-config?timeout=10s: dial tcp 192.168.100.29:6443: connect: no route to host
[reset] WARNING: Changes made to this host by 'kubeadm init' or 'kubeadm join' will be reverted.
[reset] Are you sure you want to proceed? [y/N]: y
[preflight] Running pre-flight checks
W0712 13:36:10.780087   18147 removeetcdmember.go:79] [reset] No kubeadm config, using etcd pod spec to get data directory
[reset] Stopping the kubelet service
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Deleting contents of config directories: [/etc/kubernetes/manifests /etc/kubernetes/pki]
[reset] Deleting files: [/etc/kubernetes/admin.conf /etc/kubernetes/kubelet.conf /etc/kubernetes/bootstrap-kubelet.conf /etc/kubernetes/controller-manager.conf /etc/kubernetes/scheduler.conf]
[reset] Deleting contents of stateful directories: [/var/lib/etcd /var/lib/kubelet /var/lib/dockershim /var/run/kubernetes /var/lib/cni]

The reset process does not clean CNI configuration. To do so, you must remove /etc/cni/net.d

The reset process does not reset or clean up iptables rules or IPVS tables.
If you wish to reset iptables, you must do so manually by using the "iptables" command.

If your cluster was setup to utilize IPVS, run ipvsadm --clear (or similar)
to reset your system's IPVS tables.

The reset process does not clean your kubeconfig files and you must remove them manually.
Please, check the contents of the $HOME/.kube/config file.

``` 

Lets proceed to flush iptables and delete the /etc/cni/net.d directory after confirming its content


```bash
[root@k8s-node-2 ~]# iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
[root@k8s-node-2 ~]# ls /etc/cni/net.d
10-calico.conflist  calico-kubeconfig
[root@k8s-node-2 ~]# rm -rf /etc/cni/net.d

```
