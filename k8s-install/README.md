# Up to date installation directions and manifests can be found in the [Calico docs](https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/canal)

This repo is deprecated and no further updates are expected here.

# Kubernetes self-hosted install

This directory includes manifests for deploying canal on Kubernetes using the Kubernetes API.  

#### For Kubernetes 1.7, 1.8, and 1.9

> **Note:** If you are upgrading from the Kubernetes 
[1.6](#for-kubernetes-16) or [1.5](#for-kubernetes-15) manifests to the
[1.7](#for-kubernetes-17) manifest it is neccessary to
[migrate your Calico configuration data](https://github.com/projectcalico/calico/blob/master/upgrade/v2.5/README.md)
before upgrading. Otherwise, your cluster may lose connectivity after the
upgrade.

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/rbac.yaml

kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/canal.yaml
```

#### For Kubernetes 1.6

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.6/rbac.yaml

kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.6/canal.yaml
```

#### Kubernetes 1.5

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/canal.yaml
```

### Requirements
- The Kubernetes cluster must be configured to provide serviceaccount tokens to pods.
- kubelets must be started with `--network-plugin=cni` and
  have `--cni-conf-dir` and `--cni-bin-dir` properly set
- The controller manager must be started with `--cluster-cidr=10.244.0.0/16` and `--allocate-node-cidrs=true`.

> **Note:** When using Kubernetes as the datastore the Advanced Policy
features of Calico will not be available until
[#458, Implement full policy model in kubernetes](https://github.com/projectcalico/calico/issues/458)
is finished and released.  This means it is not possible to create egress
policy for your pods, if that functionality is needed then please use etcd
as your datastore.

### Etcd

We strongly recommend using the Kubernetes API manifests above, but if you have a need to use etcd we have provided an example etcd with TLS manifest `canal_etcd_tls.yaml`.

When using an etcd datastore, the provided manifest allows you to specify the etcd endpoints for your etcd cluster,
which must be configured independently.

By default, the manifest expects an etcd proxy to be running on each Kubernetes node at `http://127.0.0.1:2379`.
