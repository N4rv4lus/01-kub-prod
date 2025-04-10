# KUBERNETES OVERVIEW
Kubernets is divided in 2 parts, the master/control plane that will manage the cluster and the nodes that will hosts the pods that will host the applications
# Master
Here is a dashboard that present the master components :

| Component                | Main Role                                                   | Managed by / Depends on        | Tool(s) Associated          | Port(s) Used              |
|--------------------------|-------------------------------------------------------------|--------------------------------|-----------------------------|---------------------------|
| kube-apiserver           | Central REST entrypoint to the cluster                      | kubelet (static pod)           | kubectl, kubeadm            | 6443 (HTTPS)              |
| kube-scheduler           | Assigns pods to nodes                                       | kubelet (static pod)           | -                           | Not exposed               |
| kube-controller-manager  | Maintains desired state (e.g. deployments, replicas, etc.)  | kubelet (static pod)           | -                           | Not exposed               |
| cloud-controller-manager | Cloud provider integration (LBs, disks, IPs, etc.)          | kubelet (static pod, optional) | -                           | Varies by cloud provider  |
| etcd                     | Key-value store of the entire cluster state                 | kubelet (static pod)           | -                           | 2379 (client), 2380 (peer)|
| Admission Controllers    | Validate/Mutate API requests                                | kube-apiserver                 | kube-apiserver              | Included in 6443          |
| CoreDNS                  | DNS for pods and services                                   | kubelet / deployment           | -                           | 53 (UDP/TCP), 9153 (metrics) |
| Metrics Server           | Exposes real-time CPU/memory metrics                        | kubelet, API Server            | metrics-server              | Not directly exposed      |
| kubeadm                  | Bootstrap/configure the Control Plane                       | Host system                    | kubeadm                     | 6443, SSH                 |


You can check in the repository how to setup a control plane in : /kubernetes/setup-control-plane 

## Workers
Here is a dashboard that present the workers components :
You can check in the repository how to setup a control plane in : /kubernetes/setup-workers (will be added in a short period)

| Component          | Main Role                                                    | Managed by / Depends on      | Tool(s) Associated          | Port(s) Used                        |
|--------------------|--------------------------------------------------------------|------------------------------|-----------------------------|-------------------------------------|
| kubelet            | Applies control plane orders locally                         | API Server                   | kubeadm, kubectl            | 10250 (HTTPS)                       |
| kube-proxy         | Manages service/pod network routing                          | iptables/IPVS, CNI           | kubeadm                     | 10256 (metrics), NodePort range 30000–32767 |
| container runtime  | Runs containers (containerd, CRI-O...)                       | kubelet via CRI              | containerd, CRI-O, Docker   | Varies (usually not exposed)       |
| CNI plugins        | Configures pod networking (overlay or direct)                | kubelet via CNI interface    | calico, flannel, cilium     | Depends on CNI (e.g. Calico: 4789 UDP / BGP: 179) |
| CSI plugins        | Manages dynamic storage provisioning                         | kubelet via CSI interface    | rook-ceph, longhorn, etc.   | Depends on plugin                  |



## Transerval Tools (clients/admin)
Here is a dashboard that present the tools from a admin/clients/user of kubernetes :
You can check in the repository how to setup a control plane in : /kubernetes/use-kubernetes (will be added in a short period)

| Component          | Main Role                                                    | Managed by / Depends on      | Tool(s) Associated          | Port(s) Used                        |
|--------------------|--------------------------------------------------------------|------------------------------|-----------------------------|-------------------------------------|
| kubelet            | Applies control plane orders locally                         | API Server                   | kubeadm, kubectl            | 10250 (HTTPS)                       |
| kube-proxy         | Manages service/pod network routing                          | iptables/IPVS, CNI           | kubeadm                     | 10256 (metrics), NodePort range 30000–32767 |
| container runtime  | Runs containers (containerd, CRI-O...)                       | kubelet via CRI              | containerd, CRI-O, Docker   | Varies (usually not exposed)       |
| CNI plugins        | Configures pod networking (overlay or direct)                | kubelet via CNI interface    | calico, flannel, cilium     | Depends on CNI (e.g. Calico: 4789 UDP / BGP: 179) |
| CSI plugins        | Manages dynamic storage provisioning                         | kubelet via CSI interface    | rook-ceph, longhorn, etc.   | Depends on plugin                  |


