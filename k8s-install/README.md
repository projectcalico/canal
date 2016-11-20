# Kubernetes self-hosted install

This directory contains Kubernetes manifests to deploy canal on Kubernetes.

- `canal.yaml`: Contains a Kubernetes DaemonSet which install and runs canal on each Kubernetes master and node.
This also includes a ReplicaSet which deploys the Calico Kubernetes policy controller, and a ConfigMap for
configuring the install.

## Configuration

### Kubernetes
Canal uses the Kubernetes API to enforce policy, and so needs to authenticate with the Kubernetes API.  The provided
ConfigMap [configures the Calico CNI plugin](https://github.com/projectcalico/calico-cni/blob/master/configuration.md#kubernetes-specific)
automatically to use serviceaccount token authentication and the Kubernetes Service clusterIP. 

### Etcd

Canal relies on etcd.  The provided manifest allows you to specify the etcd endpoints for your etcd cluster,
which must be configured independently.

By default, the provided manifest expects an etcd proxy to be running on each Kubernetes node at `http://127.0.0.1:2379`.

## Installation

To install, first make sure the ConfigMap matches your desired configuration.  Then run:

```
kubectl apply -f canal.yaml
```
