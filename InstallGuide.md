# Up to date installation directions and manifests can be found in the [Calico docs](https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/canal)

This repo is deprecated and no further updates are expected here.

# Canal Phase 1
Canal Phase 1 uses Calico to provide network policy and Flannel to provide networking between hosts.

This guide describes the parts needed to get Canal working with rkt. To try out a packaged version of this guide for Kubernetes then see the [coreos/coreos-kubernetes](https://github.com/coreos/coreos-kubernetes) repository

# flannel
flannel provides inter-host connectivity and any of the flannel networking backends can be used with Canal. The recommended choice for most deployments is `vxlan` but `host-gw` might make sense for some small clusters which are on their own L2 network.

## Configuring the cluster
Before running the flannel daemon, some global configuration needs to be written to etcd.
```
etcdctl set /coreos.com/network/config '{ "Network": "10.100.0.0/16", "Backend": {"Type": "vxlan"}}'
```

* Each host will be assigned a range of IP addresses from the network (by default a `/24`).
* Further details on the configuration can be found in the [flannel documentation](https://github.com/coreos/flannel/blob/master/README.md#configuration)

## Running the flannel daemon
Each host in the cluster needs to run the flannel daemon. If etcd is available on `localhost` then flannel doesn't need to be passed any options but typically flannel will at least need to be told how to access etcd.

flannel can be run either as a [binary](https://github.com/coreos/flannel/releases/download/v0.9.1/flanneld-amd64) or in a [container](https://quay.io/repository/coreos/flannel?tab=tags)

Some key command line options are [documented](https://github.com/coreos/flannel/blob/master/README.md#key-command-line-options)

### Example systemd unit file for running flannel under rkt
This is a simplified version of the [CoreOS](https://github.com/coreos/coreos-overlay/blob/master/app-admin/flannel/files/flanneld.service) service file. It has been tested on Ubuntu 16.04 and might need adjustment for other environments (e.g. the path to `mkdir` is different on CoreOS)
```
[Unit]
Description=Network fabric for containers
Documentation=https://github.com/coreos/flannel

[Service]
Restart=always
RestartSec=5
Environment="TMPDIR=/var/tmp/"
Environment="FLANNEL_VER=v0.9.1"
Environment="FLANNEL_IMG=quay.io/coreos/flannel"
LimitNOFILE=40000
LimitNPROC=1048576
ExecStartPre=/bin/mkdir -p /run/flannel
ExecStart=/usr/local/bin/rkt run --net=host \
   --stage1-name=coreos.com/rkt/stage1-fly \
   --insecure-options=image \
   --inherit-env=true \
   --volume runflannel,kind=host,source=/run/flannel,readOnly=false \
   --mount volume=runflannel,target=/run/flannel \
   ${FLANNEL_IMG}:${FLANNEL_VER} \
   --exec /opt/bin/flanneld \
   -- -etcd-endpoints ${ETCD_ENDPOINTS} -public-ip ${ADVERTISE_IP}

[Install]
WantedBy=multi-user.target
```
* Replace `${ETCD_ENDPOINTS}` with the location of your etcd server(s)
* Replace `${ADVERTISE_IP}` with this node's publicly routable IP.

# Running Calico on Each Host
There are two parts to using Calico to provide networking policy.
* running the `calico/node` container image on each host in your cluster.
* putting the `calico` CNI plugin on each host in your cluster.

## Calico Node
The `calico/node` container can be run using Docker or rkt. Here is an example of running it using rkt under systemd.


/etc/systemd/system/calico-node.service
````
[Unit]
Description=Calico per-host agent
Requires=network-online.target
After=network-online.target

[Service]
Slice=machine.slice
Environment=CALICO_DISABLE_FILE_LOGGING=true
Environment=NODENAME=${ADVERTISE_IP}
Environment=IP=${ADVERTISE_IP}
Environment=FELIX_FELIXHOSTNAME=${ADVERTISE_IP}
Environment=CALICO_NETWORKING=false
Environment=NO_DEFAULT_POOLS=true
Environment=ETCD_ENDPOINTS=${ETCD_ENDPOINTS}
ExecStart=/usr/bin/rkt run --inherit-env --stage1-name=coreos.com/rkt/stage1-fly \
--volume=modules,kind=host,source=/lib/modules,readOnly=false \
--mount=volume=modules,target=/lib/modules \
--trust-keys-from-https quay.io/calico/node:v2.4.1

KillMode=mixed
Restart=always
TimeoutStartSec=0

[Install]
WantedBy=multi-user.target
````
* Replace `${ETCD_ENDPOINTS}` with the location of your etcd server(s)
* Replace `${ADVERTISE_IP}` with this node's publicly routable IP.

## Calico CNI
The [Calico CNI plugin](https://github.com/projectcalico/calico-cni) is used to store information about each container that's created. This provides the `calico/node` container with the information it needs to enforce network policy.

Download the Calico CNI plugin from the [releases](https://github.com/projectcalico/calico-cni/releases) page.

When using rkt, download the CNI plugin to `/etc/rkt/net.d`

````
wget -O /etc/rkt/net.d/calico https://github.com/projectcalico/calico-cni/releases/download/v1.10.0/calico
chmod +x /etc/rkt/net.d/calico
````

## Configuring CNI
With the CNI binary in place, it's necessary to provide a configuration file. Because flannel is being used to assign an IP range to each host, we're going to use the CNI `host-local` IPAM plugin to assign addresses to containers. Normally a CNI configuration file consists of two parts 1) the networking configuration and 2) the IPAM configuration. Because we want to get the IPAM configuration from flannel, we actually need to configure flannel as the CNI plugin to call. Flannel is then told to delegate control to the Calico CNI plugin and to use `host-local` as the IPAM plugin. This process is covered in more detail in the [flannel CNI documenation](https://github.com/containernetworking/cni/blob/master/Documentation/flannel.md)

When using rkt, the file should be placed in `/etc/rkt/net.d`. rkt comes with the `flannel` and `host-local` CNI plugins built in but they can also be obtained from the [CNI release](https://github.com/containernetworking/cni/releases) page.

```json
{
    "name": "canal",
    "type": "flannel",
    "delegate": {
        "type": "calico",
        "etcd_endpoints": "${ETCD_ENDPOINTS}",
        "nodename": "${ADVERTISE_IP}"
    }
}
```
* Replace `${ADVERTISE_IP}` with this node's publicly routable IP.
* Replace `${ETCD_ENDPOINTS}` with the location of your etcd server(s)
