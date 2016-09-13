This document discusses the decisions that must be made when integrating Canal with a new orchestrator. It is intended for advanced users that are planning to do significant integration work.

Canal provides two things

- Networking between hosts in your cluster
- Enforcement of network policy

Both areas require different choices for orchestrator integrators and are covered in detail below.

# Networking
Networking can be handled either by `flannel` or by `calico`. Regardless of which is used, Calico needs to be involved when containers are started. See the [Policy](#policy) section below. 

## Flannel
If flannel is being used to provide inter-host connectivity then any of the flannel backends can be used. To set up flannel follow the [instructions](https://coreos.com/flannel/docs/latest/flannel-config.html).

Flannel can run as a native binary or from inside a container.

### IPAM
When Flannel is being used to provide connectivity, then Calico should not be used for IPAM. Flannel requires IP addresses to be allocated only from a range assigned to that host and Calico IPAM doesn't currently support this mode. 

## Calico
If Calico is being used for networking then "Canal" is precisely the same as "Calico". Full instructions for setting up Calico networking can be found in the Calico [documentation](https://github.com/projectcalico/calico-containers/blob/master/README.md)

Briefly, it involves
* Running a BGP daemon (bird) on each host
* Storing configuration for the BGP daemon in the etcd datastore.
* Running Confd as a glue layer to translate the etcd data into valid bird configuration.

This functionality is packaged into the `calico/node` container image. An example of running it under rkt can be found in the [coreos-kubernetes](https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/deploy-master.md#set-up-calico-node-container-optional) repository.
The configuration can be written to etcd using the [`calicoctl`](https://github.com/projectcalico/calico-containers/tree/v0.22.0/docs/calicoctl.md) CLI tool, one of the Calico client libraries or it can be written to [etcd directly](https://github.com/projectcalico/calico-containers/blob/v0.22.0/docs/etcdStructure.md)
 
### IPAM
When Calico is being used to provide connectivity, then Calico should be used for IPAM. Using Calico for IPAM ensures that addresses are allocated in such a way to improve aggregation for BGP. 
 
# Policy
Calico enforces policy by watching etcd for policy definitions and "endpoints" and rendering them to `iptables` rules on each host. Endpoints are the networking interfaces inside containers. The component that does this watching is called `Felix` 

## Deploying Felix
There are a number of options for deploying Felix, including running the binary directly or running it in a container. The Felix [project page](https://github.com/projectcalico/calico/blob/1.4.1b2/README.md#how-do-i-buildrun-Felix) details the options for running Felix by itself.

The easiest way to run Felix is using the `calico/node` container alongside the `bird` BGP agent.
 
## Storing "endpoints" in etcd
For `Felix` to work, the details of each container need to be written to etcd. This can be done using plugins to the container runtime or orchestrator
* Using the [Calico CNI plugin](https://github.com/projectcalico/calico-cni) e.g. on `rkt`, `Kubernetes` or `Mesos`
* Using the [Calico libnetwork plugin](https://github.com/projectcalico/libnetwork-plugin) e.g. for `Docker`

Or it can be done by writing the information directly to etcd. There are libraries to help with this
* Python - https://github.com/projectcalico/libcalico
* Go - https://github.com/tigera/libcalico-go

### Using CNI with Flannel
When using Flannel for networking and using Calico CNI to store the endpoints in etcd, the flannel CNI plugin should be used to wrap the Calico CNI plugin. This allows the IP range that flannel allocated to the host to be passed to the `host-local` IPAM plugin. An example of this is provided in the [coreos-kubernetes](https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/deploy-master.md#set-up-the-cni-config-optional) repository.
