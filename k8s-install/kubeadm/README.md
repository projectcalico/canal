# Install for Kubeadm

This directory contains a single packaged manfest for installing Canal on kubeadm managed Kubernetes clusters.  It is a specific case of the 
more general manifests provided [here](../README.md)

To install this manifest, make sure you've created a cluster using the kubeadm tool.

Then use kubectl to create the manifest in this directory:

```
kubectl create -f canal.yaml
```

## About

This manifest deploys the standard Canal components described [here](../README.md) as well as a dedicated etcd
node on the Kubernetes master.  Note that in a production cluster, it is recommended you use a secure, replicated etcd cluster.

### Requirements / Limitations

* This manifest requires the ability to schedule pods on the master.  As such, make sure you initialize your master with `kubeadm init --schedule-workload`

