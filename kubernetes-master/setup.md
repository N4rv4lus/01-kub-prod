We'll see how to setup a "secured" kubernetes environnement. (This is a base setup that will improved, you can always add all lot of configurations to make it more secured because there are always new CVE)

## Ports & Nftables
On debian 12.10 iptables is deprecated, and the os is using it's "big brother", nftables.
So here setup your nftables configuration.

We need to let open theses specifics ports for kubernetes :
- TCP	Inbound	6443	Kubernetes API server	All
- TCP	Inbound	2379-2380	etcd server client API	kube-apiserver, etcd
- TCP	Inbound	10250	Kubelet API	Self, Control plane
- TCP	Inbound	10259	kube-scheduler	Self
- TCP	Inbound	10257	kube-controller-manager	Self
- UDP   Inbound 4789    CNI for calico vxlan
- UDP   Outound 4789    CNI for calico vxlan

and we also need theses ports for administration :
- 22 for ssh access
- 53 TCP & UDP for DNS
- 9100 for node-exporter

## Containerd

install and configure config.toml