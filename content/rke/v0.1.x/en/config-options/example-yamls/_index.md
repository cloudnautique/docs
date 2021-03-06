---
title: Example Cluster.ymls
weight: 3000
draft: true
---


### Minimal `cluster.yml` example

```
# default k8s version: v1.8.10-rancher1-1
# default network plugin: canal
nodes:
  - address: 1.2.3.4
    user: ubuntu
    role: [controlplane,worker,etcd]
```

### Full `cluster.yml` example

```yaml
---
nodes:
  - address: 1.1.1.1
    user: ubuntu
    role:
    - controlplane
    - etcd
    ssh_key_path: /home/user/.ssh/id_rsa
    port: 2222
  - address: 2.2.2.2
    user: ubuntu
    role:
    - worker
    ssh_key: |-
      -----BEGIN RSA PRIVATE KEY-----

      -----END RSA PRIVATE KEY-----
  - address: example.com
    user: ubuntu
    role:
    - role
    hostname_override: node3
    internal_address: 192.168.1.6
    labels:
      app: ingress

services:
  etcd:
    # if external etcd is used
    # path: /etcdcluster
    # external_urls:
    #   - https://etcd-example.com:2379
    # ca_cert: |-
    #   -----BEGIN CERTIFICATE-----
    #   xxxxxxxxxx
    #   -----END CERTIFICATE-----
    # cert: |-
    #   -----BEGIN CERTIFICATE-----
    #   xxxxxxxxxx
    #   -----END CERTIFICATE-----
    # key: |-
    #   -----BEGIN PRIVATE KEY-----
    #   xxxxxxxxxx
    #   -----END PRIVATE KEY-----
  kube-api:
    service_cluster_ip_range: 10.43.0.0/16
    pod_security_policy: false
    # add additional arguments to the kubernetes component
    # Note that this WILL OVERRIDE existing defaults
    extra_args:
      # Enable audit log to stdout
      audit-log-path: "-"
      # Increase number of delete workers
      delete-collection-workers: 3
      # Set the level of log output to debug-level
      v: 4
  kube-controller:
    cluster_cidr: 10.42.0.0/16
    service_cluster_ip_range: 10.43.0.0/16
  scheduler:
  kubelet:
    cluster_domain: cluster.local
    cluster_dns_server: 10.43.0.10
    infra_container_image: gcr.io/google_containers/pause-amd64:3.0
    # Optionally define additional volume binds to a service
    extra_binds:
      - "/usr/libexec/kubernetes/kubelet-plugins:/usr/libexec/kubernetes/kubelet-plugins"
  kubeproxy:

# supported plugins are:
# flannel
# calico
# canal
# weave
#
# If you are using calico on AWS or GCE, use the network plugin config option:
# 'calico_cloud_provider: aws'
# or
# 'calico_cloud_provider: gce'
# network:
#   plugin: calico
#   options:
#     calico_cloud_provider: aws
#
# To specify flannel interface, you can use the 'flannel_iface' option:
# network:
#   plugin: flannel
#   options:
#     flannel_iface: eth1
# To specify flannel interface for canal plugin, you can use the 'canal_iface' option:
# network:
#   plugin: canal
#   options:
#     canal_iface: eth1


network:
  plugin: flannel
  options:

# At the moment, the only authentication strategy supported is x509.
# You can optionally create additional SANs (hostnames or IPs) to add to
#  the API server PKI certificate. This is useful if you want to use a load balancer
#  for the control plane servers, for example.
authentication:
  strategy: x509
  sans:
  - "10.18.160.10"
  - "my-loadbalancer-1234567890.us-west-2.elb.amazonaws.com"

# all addon manifests MUST specify a namespace
addons: |-
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: my-nginx
      namespace: default
    spec:
      containers:
      - name: my-nginx
        image: nginx
        ports:
        - containerPort: 80

addons_include:
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-operator.yaml
    - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-cluster.yaml
    - /path/to/manifest

system_images:
  etcd: rancher/etcd:v3.0.17
  kubernetes: rancher/hyperkube:v1.10.1
  alpine: rancher/rke-tools:v0.1.4
  nginx_proxy: rancher/rke-tools:v0.1.4
  cert_downloader: rancher/rke-tools:v0.1.4
  kubernetes_services_sidecar: rancher/rke-tools:v0.1.4
  kubedns: rancher/k8s-dns-kube-dns-amd64:1.14.5
  dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.14.5
  kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.14.5
  kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0
  flannel: rancher/coreos-flannel:v0.9.1
  flannel_cni: rancher/coreos-flannel-cni:v0.2.0


ssh_key_path: ~/.ssh/test

# Kubernetes authorization mode
# Use `mode: rbac` to enable RBAC
# Use `mode: none` to disable authorization
authorization:
  mode: rbac

# If set to true, rke won't fail when unsupported Docker version is found
ignore_docker_version: false
# The kubernetes version used. For now, this should match the version defined in rancher/types defaults map: https://github.com/rancher/types/blob/master/apis/management.cattle.io/v3/k8s_defaults.go#L14
kubernetes_version: v1.10.1-rancher1

# addons are deployed using kubernetes jobs. RKE will give up on trying to get the job status after this timeout in seconds..
addon_job_timeout: 30
# If set, this is the cluster name that will be used in the kube config file
# Default value is "local"
cluster_name: mycluster

# List of registry credentials, if you are using a Docker Hub registry,
# you can omit the `url` or set it to `docker.io`
private_registries:
  - url: registry.com
    user: Username
    password: password

# Currently only nginx ingress provider is supported.
# To disable ingress controller, set `provider: none`
# To enable ingress on specific nodes, use the node_selector, eg:
# nodes:
#   - address: example.com
#     user: ubuntu
#     role:
#     - role
#     hostname_override: node3
#     internal_address: 192.168.1.6
#     labels:
#       app: ingress
#
# ingress:
#   provider: nginx
#   node_selector:
#     app: ingress
#   extra_args:
#     enable-ssl-passthrough: ""

ingress:
  provider: nginx

cloud_provider:
  name: aws

# Bastion/Jump host configuration
bastion_host:
  address: x.x.x.x
  user: ubuntu
  port: 22
  ssh_key_path: /home/user/.ssh/bastion_rsa
  # or
  # ssh_key: |-
  #   -----BEGIN RSA PRIVATE KEY-----
  #
  #   -----END RSA PRIVATE KEY-----
  ```
