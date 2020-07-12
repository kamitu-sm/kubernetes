# Adding worker node #

For each additional worker node you should execute the join command that was previously given to you by the kubeadm init output on the first node. It should look something like this:

```
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
    --discovery-token-ca-cert-hash sha256:b17c8c854ddb9e5cc514c0a607aab57ae640a26ffeff8c0ebf390420aee71f43 
```

So we are following this, the discovery token has changed since i had to reinistiaslise the cluster after it constantly crushed as a result of running the ***kubeadm reset*** command after deleting the secondary master node 

```bash
[root@k8s-node-2 ~]# kubeadm join k8s-api-lb:6443 --token y40psg.1gl8f0exzc11pedz \
>     --discovery-token-ca-cert-hash sha256:08591777d66cd9abaedb595708db939c995638df6acc761f8f5d9502740b4541
W0712 15:27:55.736667    3958 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING FileExisting-tc]: tc not found in system path
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

```

On the master

```bash
[stephen@k8s-master-1 ~]$ kubectl get nodes
NAME           STATUS   ROLES    AGE    VERSION
k8s-master-1   Ready    master   11m    v1.18.5
k8s-node-2     Ready    <none>   8m7s   v1.18.5
[stephen@k8s-master-1 ~]$ kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE     IP               NODE           NOMINATED NODE   READINESS GATES
kube-system   calico-kube-controllers-76d4774d89-s5x9k   1/1     Running   0          10m     10.244.196.1     k8s-master-1   <none>           <none>
kube-system   calico-node-7kt7d                          0/1     Running   0          10m     192.168.100.29   k8s-master-1   <none>           <none>
kube-system   calico-node-nhg8s                          0/1     Running   0          8m19s   192.168.100.35   k8s-node-2     <none>           <none>
kube-system   coredns-66bff467f8-99nss                   1/1     Running   0          11m     10.244.196.2     k8s-master-1   <none>           <none>
kube-system   coredns-66bff467f8-zkgw9                   1/1     Running   0          11m     10.244.196.3     k8s-master-1   <none>           <none>
kube-system   etcd-k8s-master-1                          1/1     Running   0          11m     192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-apiserver-k8s-master-1                1/1     Running   0          11m     192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-controller-manager-k8s-master-1       1/1     Running   0          11m     192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-proxy-56npp                           1/1     Running   0          8m19s   192.168.100.35   k8s-node-2     <none>           <none>
kube-system   kube-proxy-bgjhn                           1/1     Running   0          11m     192.168.100.29   k8s-master-1   <none>           <none>
kube-system   kube-scheduler-k8s-master-1                1/1     Running   0          11m     192.168.100.29   k8s-master-1   <none>           <none>
[stephen@k8s-master-1 ~]$ 
```

