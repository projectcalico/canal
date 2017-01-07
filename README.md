
![Canal Logo](logos/canal-logo-type-full-color 328x184.png)

# Policy based networking for cloud native applications

Canal is a community-driven initiative that aims to allow users to easily deploy Calico and flannel networking together as a unified networking solution - combining Calico’s industry-leading network policy enforcement with the rich superset of Calico and flannel overlay and non-overlay network connectivity options.

Canal represents the best-of-breed policy-based networking solution for cloud native applications, supporting any orchestrator that support the CNI network plugin API (including Kubernetes, Mesos, and others).

![Canal Diagram](./Canal Phase 1 Diagram.png)

Note that the Canal currently uses the Calico and flannel projects as is with no code modifications to either. Canal today is simply a deployment pattern for installing and configuring the projects to work together seamlessly as single network solution from the point of view of the user and orchestration system. In the future the Canal project will likely contribute code changes to Calico and flannel projects to further simplify install and configuration.

## Canal installation instructions

### CoreOS based Kubernetes cluster
Canal is supported today within the [coreos/coreos-kubernetes](https://github.com/coreos/coreos-kubernetes) repository, which provides step-by-step instructions plus several automated install options to bring up a Kubernetes cluster with Canal networking (Calico policy + flannel VXLAN connectivity by default).

### Kubernetes self-hosted install
[See here](k8s-install/README.md)

[See here](k8s-install/kubeadm/README.md) for a version specifically for `kubeadm` clusters.

### Step-by-step guide for expert rkt users
[See here](InstallGuide.md)

### Step-by-step guide for expert users creating their own orchestrator integrations
[See here](OrchestratorIntegration.md)

### Other automated install options
Watch this space for news of further integration and install instructions coming soon!

## AWS Reference Architectures

There are essentially three options for AWS networking: VXLAN encap, IPIP encap, and native VPC routing. This will walk through the pros/cons and recommendations.

### Flannel VXLAN

*This is our default recommendation for 2017 Q1.*

Flannel's VXLAN provides simplicity and performance both inside of an AWS VPC and across VPCs. And with the recently merged support to deploy flannel simply using [kubectl](https://github.com/coreos/flannel/issues/587) it is easy to get started as well. For users this is a reasonable default recommended by teams from both CoreOS and Tigera.

**Note:** The AWS VPC will have no ability to enforce routing policy at the Pod level, only at the node level.

### Calico Linux Routing Inside a AZ and IPIP Encap Cross AZ (Q2 2017)

*This will be our default recommendation once completed.*

Because Amazon VPCs are L2 networks Calico can use native Linux routing between hosts in a VPC AZ / subnet. This means that packets flow between hosts using the normal Linux routing logic.

However, Amazon does L3 dest filtering between VPC AZ subnets and so for all routes outside of a VPC subnet Calico will do IPIP encapsulation. IPIP encap has advantages over VXLAN: no L2 announces to track/refresh, packets keep routing with control plane down, slightly fewer bytes for encap.

Customer VPCs must be configured with the correct settings on for the intra-vpc routing to work correctly (see Calico AWS docs: "Routing Traffic Within a Single VPC Subnet" http://docs.projectcalico.org/v2.0/reference/public-cloud/aws#routing-traffic-within-a-single-vpc-subnet).

**Note:** The AWS VPC will have no ability to enforce routing policy at the Pod level, only at the node level.

### Native AWS VPC Routing

*We do not recommend this configuration.*

AWS VPC does have the ability to setup native routes which could be used to setup the Pod networking. However, VPC puts extremely low limits on these route entries (50 per VPC). Because of this limited scalability it is not a recommended configuration.

## Roadmap
- [x] Automated and manual install instructions for CoreOS based Kubernetes clusters
- [x] Step-by-step guide for users creating their own install solutions
- [x] Kubernetes self-hosted installation support (installing Calico & flannel in a single Kubernetes pod, run as a Daemonset)
- [x] Kubernetes self-hosted etcd-less installation support (Calico & flannel using k8s API server only, without direct etcd access)
- [ ] Combine Calico and Flannel into single container
- [ ] Single CNI plugin with simplified configuration
- [ ] Mesos and DC/OS universe installation support
- [ ] Combine Calico and Flannel daemons into single daemon
