# Install for Kubeadm

This directory contains a single packaged manifest for installing Canal on kubeadm managed Kubernetes clusters.  It is a specific case of the
more general manifests provided [here](../README.md)

To install this manifest, make sure you've created a cluster using the kubeadm tool.

Then use kubectl to create the manifest in this directory depending on your Kubernetes version.

For Kuberentes 1.5 and below:

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/kubeadm/canal.yaml
```

For Kubernetes 1.6:

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/kubeadm/1.6/canal.yaml
```

## About

This manifest deploys the standard Canal components described [here](../README.md) as well as a dedicated etcd
node on the Kubernetes master.  Note that in a production cluster, it is recommended you use a secure, replicated etcd cluster.

### Requirements / Limitations

* This install does not configure etcd TLS
* This install expects that your Kubernetes master node has been labeled with:
  * Kubernetes 1.5: `kubeadm.alpha.kubernetes.io/role: master`
  * Kubernetes 1.6: `node-role.kubernetes.io/master`
