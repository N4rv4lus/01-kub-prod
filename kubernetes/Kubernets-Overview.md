# KUBERNETES OVERVIEW
Kubernets is divided in 2 parts, the master/control plane that will manage the cluster and the nodes that will hosts the pods that will host the applications
# Master
Here is a dashboard that present the master components :

| Composant                 | Rôle principal                                               | Géré par / dépend de             | Outil(s) associé(s)                  |
|---------------------------|--------------------------------------------------------------|----------------------------------|--------------------------------------|
| kube-apiserver            | Point d’entrée REST du cluster                               | kubelet (pod statique)           | kubectl, kubeadm                     |
| kube-scheduler            | Assigne les pods aux nodes                                   | kubelet (pod statique)           | -                                    |
| kube-controller-manager   | Maintient l’état désiré du cluster                           | kubelet (pod statique)           | -                                    |
| cloud-controller-manager  | Intégration avec les cloud providers (LB, volumes, etc.)     | kubelet (pod statique, optionnel)| -                                    |
| etcd                      | Base de données clé/valeur du cluster                        | kubelet (pod statique)           | -                                    |
| Admission Controllers     | Valident ou modifient les requêtes à l’API Server            | kube-apiserver                   | Flags dans kube-apiserver            |
| CoreDNS                   | DNS interne pour les pods et services                        | kubelet / déploiement            | -                                    |
| Metrics Server            | Fournit les métriques système aux autoscalers                | kubelet, API Server              | metrics-server                       |
| kubeadm                   | Installation/configuration du Control Plane                  | Système hôte                     | kubeadm                              |

You can check in the repository how to setup a control plane in : /kubernetes/setup-control-plane (will be added in a short period)

## Workers
Here is a dashboard that present the workers components :

| Composant          | Rôle principal                                              | Géré par / dépend de          | Outil(s) associé(s)                    |
|--------------------|-------------------------------------------------------------|-------------------------------|----------------------------------------|
| kubelet            | Exécute localement les ordres du Control Plane              | API Server                    | kubeadm, kubectl                       |
| kube-proxy         | Gère le routage réseau entre services/pods                  | iptables/IPVS, CNI            | kubeadm                                |
| container runtime  | Exécute les containers (containerd, CRI-O...)               | kubelet via CRI               | containerd, CRI-O, Docker (deprecated) |
| CNI plugins        | Configure le réseau des pods (overlay ou non)               | kubelet via interface CNI     | calico, flannel, cilium, etc.          |
| CSI plugins        | Gère les volumes de stockage (dynamic provisioning)         | kubelet via interface CSI     | rook-ceph, longhorn, etc.              |

You can check in the repository how to setup a control plane in : /kubernetes/setup-workers

## Transerval Tools (clients/admin)
Here is a dashboard that present the tools from a admin/clients/user of kubernetes :

| Outil        | Fonction                                                     | Utilisé avec / dépend de        |
|--------------|--------------------------------------------------------------|---------------------------------|
| kubectl      | Interface CLI principale pour interagir avec l’API Server    | kube-apiserver                  |
| kubeadm      | Bootstrap et configuration du cluster Kubernetes             | Nodes, API Server               |
| helm         | Gestionnaire de packages Kubernetes (charts)                 | kube-apiserver                  |
| kustomize    | Générateur de manifestes Kubernetes personnalisés            | kubectl, CI/CD                  |

You can check in the repository how to setup a control plane in : /kubernetes/use-kubernetes (will be added in a short period)