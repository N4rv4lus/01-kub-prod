We'll see how to setup a "secured" kubernetes environnement.
So here setup your nftables configuration.

We need to let open theses specifics ports for kubernetes :
- TCP	Inbound	6443	Kubernetes API server	All
- TCP	Inbound	2379-2380	etcd server client API	kube-apiserver, etcd
- TCP	Inbound	10250	Kubelet API	Self, Control plane
- TCP	Inbound	10259	kube-scheduler	Self
- TCP	Inbound	10257	kube-controller-manager	Self

and we also need theses ports for administration :
- 22 for ssh access
- 53 TCP & UDP for DNS
- 9100 for node-exporter