We'll see how to setup a "secured" kubernetes environnement. (This is a base setup that will improved, you can always add all lot of configurations to make it more secured because there are always new CVE).
All the servers OS are on debian 12.10

# Hostname & DNS

Setup you hostname : 
```shell
hostnamectl KubMaster00
```

Add your hostname to the forward and reverse zone in your dns.

# Ports & Nftables
On debian 12.10 iptables is deprecated, and the os is using it's "big brother", nftables.
So here setup your nftables configuration.

## Kubernetes and administration Ports
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

## Configuration
Now edit nftables configuration file (check nftables.conf in the repo):
```shell
nano /etc/nftables.conf
```
And then apply it and check the status :
```shell
systemctl restart nftables && systemctl status nftables
```
The output should be something like this :
```shell
● nftables.service - nftables
     Loaded: loaded (/lib/systemd/system/nftables.service; enabled; preset: enabled)
     Active: active (exited) since Sat 2025-04-05 17:07:19 CEST; 3ms ago
       Docs: man:nft(8)
             http://wiki.nftables.org
    Process: 2034 ExecStart=/usr/sbin/nft -f /etc/nftables.conf (code=exited, status=0/SUCCESS)
   Main PID: 2034 (code=exited, status=0/SUCCESS)
        CPU: 7ms

Apr 05 17:07:19 KubMaster00 systemd[1]: nftables.service: Deactivated successfully.
Apr 05 17:07:19 KubMaster00 systemd[1]: Stopped nftables.service - nftables.
Apr 05 17:07:19 KubMaster00 systemd[1]: Starting nftables.service - nftables...
Apr 05 17:07:19 KubMaster00 systemd[1]: Finished nftables.service - nftables.
```
If you have an error check the journal :
```shell
journalctl -xeu nftables
```
# Kernel network setup

## /etc/sysctl.d

As mentionned in kubernetes documentation enable IPv4 packet forwarding for CNI : (https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
```shell
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
```
Apply sysctl system new parameters with :
```shell
sudo /usr/sbin/sysctl --system
```
And confirm the modification, the output should have this line value at 1 "net.ipv4.ip_forward = 1" :
```shell
* Applying /usr/lib/sysctl.d/50-pid-max.conf ...
* Applying /usr/lib/sysctl.d/99-protect-links.conf ...
* Applying /etc/sysctl.d/99-sysctl.conf ...
* Applying /etc/sysctl.d/k8s.conf ...
* Applying /etc/sysctl.conf ...
kernel.pid_max = 4194304
fs.protected_fifos = 1
fs.protected_hardlinks = 1
fs.protected_regular = 2
fs.protected_symlinks = 1
net.ipv4.ip_forward = 1
```
You can also check the only line with :
```shell
/usr/sbin/sysctl net.ipv4.ip_forward
```



# Containerd


## Containerd installation
Now install containerd and runc :
```shell
apt install -y containerd runc
```
As you will see if you run this command to check containerd configuration file : /etc/containerd/config.toml the file do not have much parameter,
```shell
cat /etc/containerd/config.toml
```
Here is the output :
```shell
version = 2

[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/usr/lib/cni"
      conf_dir = "/etc/cni/net.d"
  [plugins."io.containerd.internal.v1.opt"]
    path = "/var/lib/containerd/opt"
```
But you want to enable the default configuration and then configure it as you want for your specifications.
So run : 
```shell
containerd config default > /etc/containerd/config.toml
```
This should not output anything, and now you can check the default configuration for containerd.
```shell
disabled_plugins = []
imports = []
oom_score = 0
plugin_dir = ""
required_plugins = []
root = "/var/lib/containerd"
state = "/run/containerd"
temp = ""
version = 2

[cgroup]
  path = ""

[debug]
  address = ""
  format = ""
  gid = 0
  level = ""
  uid = 0

[grpc]
  address = "/run/containerd/containerd.sock"
  gid = 0
  max_recv_message_size = 16777216
  max_send_message_size = 16777216
  tcp_address = ""
  tcp_tls_ca = ""
  tcp_tls_cert = ""
  tcp_tls_key = ""
  uid = 0

[metrics]
  address = ""
  grpc_histogram = false

[plugins]

  [plugins."io.containerd.gc.v1.scheduler"]
    deletion_threshold = 0
    mutation_threshold = 100
    pause_threshold = 0.02
    schedule_delay = "0s"
    startup_delay = "100ms"

  [plugins."io.containerd.grpc.v1.cri"]
    device_ownership_from_security_context = false
    disable_apparmor = false
    disable_cgroup = false
    disable_hugetlb_controller = true
    disable_proc_mount = false
    disable_tcp_service = true
    enable_selinux = false
    enable_tls_streaming = false
    enable_unprivileged_icmp = false
    enable_unprivileged_ports = false
    ignore_image_defined_volumes = false
    max_concurrent_downloads = 3
    max_container_log_line_size = 16384
    netns_mounts_under_state_dir = false
    restrict_oom_score_adj = false
    sandbox_image = "registry.k8s.io/pause:3.6"
    selinux_category_range = 1024
    stats_collect_period = 10
    stream_idle_timeout = "4h0m0s"
    stream_server_address = "127.0.0.1"
    stream_server_port = "0"
    systemd_cgroup = false
    tolerate_missing_hugetlb_controller = true
    unset_seccomp_profile = ""

    [plugins."io.containerd.grpc.v1.cri".cni]
      bin_dir = "/opt/cni/bin"
      conf_dir = "/etc/cni/net.d"
      conf_template = ""
      ip_pref = ""
      max_conf_num = 1

    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "runc"
      disable_snapshot_annotations = true
      discard_unpacked_layers = false
      ignore_rdt_not_enabled_errors = false
      no_pivot = false
      snapshotter = "overlayfs"

      [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            BinaryName = ""
            CriuImagePath = ""
            CriuPath = ""
            CriuWorkPath = ""
            IoGid = 0
            IoUid = 0
            NoNewKeyring = false
            NoPivotRoot = false
            Root = ""
            ShimCgroup = ""
            SystemdCgroup = false

      [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
        base_runtime_spec = ""
        cni_conf_dir = ""
        cni_max_conf_num = 0
        container_annotations = []
        pod_annotations = []
        privileged_without_host_devices = false
        runtime_engine = ""
        runtime_path = ""
        runtime_root = ""
        runtime_type = ""

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

    [plugins."io.containerd.grpc.v1.cri".image_decryption]
      key_model = "node"

    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]

      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

    [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
      tls_cert_file = ""
      tls_key_file = ""

  [plugins."io.containerd.internal.v1.opt"]
    path = "/opt/containerd"

  [plugins."io.containerd.internal.v1.restart"]
    interval = "10s"

  [plugins."io.containerd.metadata.v1.bolt"]
    content_sharing_policy = "shared"

  [plugins."io.containerd.monitor.v1.cgroups"]
    no_prometheus = false

  [plugins."io.containerd.runtime.v1.linux"]
    no_shim = false
    runtime = "runc"
    runtime_root = ""
    shim = "containerd-shim"
    shim_debug = false

  [plugins."io.containerd.runtime.v2.task"]
    platforms = ["linux/amd64"]
    sched_core = false

  [plugins."io.containerd.service.v1.diff-service"]
    default = ["walking"]

  [plugins."io.containerd.service.v1.tasks-service"]
    rdt_config_file = ""

  [plugins."io.containerd.snapshotter.v1.aufs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.btrfs"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.devmapper"]
    async_remove = false
    base_image_size = ""
    discard_blocks = false
    fs_options = ""
    fs_type = ""
    pool_name = ""
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.native"]
    root_path = ""

  [plugins."io.containerd.snapshotter.v1.overlayfs"]
    root_path = ""
    upperdir_label = false

  [plugins."io.containerd.snapshotter.v1.zfs"]
    root_path = ""

[proxy_plugins]

[stream_processors]

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar"

  [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
    accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
    args = ["--decryption-keys-path", "/etc/containerd/ocicrypt/keys"]
    env = ["OCICRYPT_KEYPROVIDER_CONFIG=/etc/containerd/ocicrypt/ocicrypt_keyprovider.conf"]
    path = "ctd-decoder"
    returns = "application/vnd.oci.image.layer.v1.tar+gzip"

[timeouts]
  "io.containerd.timeout.bolt.open" = "0s"
  "io.containerd.timeout.shim.cleanup" = "5s"
  "io.containerd.timeout.shim.load" = "5s"
  "io.containerd.timeout.shim.shutdown" = "3s"
  "io.containerd.timeout.task.state" = "2s"

[ttrpc]
  address = ""
  gid = 0
  uid = 0
```
So to enable containerd to have access to specific hardware functionnality (cpu/ram/storage/network interface) for pods, you will have to permit it to have access cgroup :
modify this line in /etc/containerd/config.toml
```conf
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```
You also need to check that CRI is not in the disabled_plugins so check with :
```shell
cat /etc/containerd/config.toml | grep disabled_plugins
```
The output should be : 
```shell
disabled_plugins = []
```
Now restart containerd and check your containerd status : 
```shell
systemctl restart containerd && systemctl status containerd
```
here the output :
```shell
● containerd.service - containerd container runtime
     Loaded: loaded (/lib/systemd/system/containerd.service; enabled; preset: enabled)
     Active: active (running) since Sat 2025-04-05 19:00:50 CEST; 3ms ago
       Docs: https://containerd.io
    Process: 2347 ExecStartPre=/sbin/modprobe overlay (code=exited, status=0/SUCCESS)
   Main PID: 2349 (containerd)
      Tasks: 14
     Memory: 15.7M
        CPU: 33ms
     CGroup: /system.slice/containerd.service
             └─2349 /usr/bin/containerd

Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366548260+02:00" level=info msg="Start recovering state"
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366520056+02:00" level=info msg=serving... address=/run/containerd/debug.sock
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366609516+02:00" level=info msg=serving... address=/run/containerd/containerd.sock.ttrpc
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366609657+02:00" level=info msg="Start event monitor"
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366700238+02:00" level=info msg="Start snapshots syncer"
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366709837+02:00" level=info msg="Start cni network conf syncer for default"
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366662537+02:00" level=info msg=serving... address=/run/containerd/containerd.sock
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366714576+02:00" level=info msg="Start streaming server"
Apr 05 19:00:50 KubMaster00 containerd[2349]: time="2025-04-05T19:00:50.366772666+02:00" level=info msg="containerd successfully booted in 0.009104s"
Apr 05 19:00:50 KubMaster00 systemd[1]: Started containerd.service - containerd container runtime.
```

## Disable Swap
comment fstab line to disable swap

## Install Kubelet / Kubeadm / Kubectl


## Setup Cluster with KUBEADM 

## Install and Configure CNI - Calico