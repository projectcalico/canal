# Kubernetes self-hosted install

This directory contains Kubernetes manifests to deploy canal on Kubernetes.

- `config.yaml`: Contains a Kubernetes ConfigMap for configuring the deployment.  Make sure the values
in this file match your desired configuration.

- `canal.yaml`: Contains a Kubernetes DaemonSet which install and runs canal on each Kubernetes master and node.
This also includes a ReplicaSet which deploys the Calico Kubernetes policy controller.

## Configuration

### Kubernetes
Canal uses the Kubernetes API to enforce policy, and so needs to authenticate with the Kubernetes API.  The provided
`config.yaml` [configures the Calico CNI plugin](https://github.com/projectcalico/calico-cni/blob/master/configuration.md#kubernetes-specific)
automatically to use serviceaccount token authentication and the Kubernetes Service clusterIP. 

### Etcd

Canal relies on etcd.  The provided `config.yaml` allows you to specify the etcd endpoints for your etcd cluster,
which must be configured independently.

By default, the provided manifest expects an etcd proxy to be running on each Kubernetes node at `http://127.0.0.1:2379`.

## Installation

To install, first make sure `config.yaml` matches your desired configuration.  Then run:

```
kubectl apply -f config.yaml -f canal.yaml
```
