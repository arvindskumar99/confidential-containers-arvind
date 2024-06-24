# SEV-ES guide with no pre-attestation

This guide covers platform-specific setup for SEV(-ES) and walks through
complete flows to launch COCO with no pre-attestation. This guide assumes 
that you've already installed the [snp.sh](https://github.com/amd/sev-utils/tree/main/tools) 
script, a tool that builds the required patched versions of qemu ovmf, and a linux kernel, 
tools you will need to create your COCO instance without pre-attestation.

# Installation

You can enable Confidential Containers in an existing Kubernetes cluster using the Confidential Containers Operator.
When installation is finished, your cluster will have new runtime classes for different hardware platforms,
including a generic runtime for testing CoCo without confidential hardware support, a runtime using a remote hypervisor
that allows for cloud integration, a runtime for process-based isolation using SGX, as well as runtimes for TDX and SEV.

## Prerequisites

To run the operator you must have an existing Kubernetes cluster that meets the followng requirements.

- Ensure a minimum of 8GB RAM and 4 vCPU for the Kubernetes cluster node
- Only containerd runtime based Kubernetes clusters are supported with the current CoCo release
- The minimum Kubernetes version should be 1.24
- Ensure at least one Kubernetes node in the cluster is having the label `node-role.kubernetes.io/worker=` and `node.kubernetes.io/worker=`
- Ensure SELinux is disabled or not enforced (https://github.com/confidential-containers/operator/issues/115)
- Ensure Docker is installed. Instructions can be found [here](https://docs.docker.com/engine/install/ubuntu/). Make sure to also install docker-compose with docker.

For more details on the operator, including the custom resources managed by the operator, refer to the operator [docs](https://github.com/confidential-containers/operator).

# Operator Installation

## Get environment ready for operator deployment.

Please refer to the prerequisites section to ensure your Kubernetes cluster meets the requirements. 

When adding the worker label to your node, make sure to add both of the following annotations.

```
kubectl label node "${NODE_NAME}" node.kubernetes.io/worker= 2>/dev/null || true

kubectl label node "${NODE_NAME}" node-role.kubernetes.io/worker= 2>/dev/null || true
```

To make sure SELinux is disabled use the command 

## Deploy the operator
Deploy the operator by running the following command, replace `<RELEASE_VERSION>` with the desired version of the operator. The available versions can be found here: [release tag](https://github.com/confidential-containers/operator/tags)

```
kubectl apply -k github.com/confidential-containers/operator/config/release?ref=<RELEASE_VERSION>
```

The latest published release is v0.8.0. To deploy an operator with v0.8.0, use the following command.

```
kubectl apply -k github.com/confidential-containers/operator/config/release?ref=v0.8.0
```

Use the following command to monitor the status of each pod. Wait until each pod has a status of running.

```
kubectl get pods -n confidential-containers-system --watch
```

## Create the custom resource

Creating a custom resource installs the required CC runtime pieces into teh cluster node and creates the `RuntimeClasses`.

```
kubectl apply -k github.com/confidential-containers/operator/config/samples/ccruntime/<CCRUNTIME_OVERLAY>?ref=<RELEASE_VERSION>
```

The current present overlays are: `default` and `s390x`.

```
kubectl apply -k github.com/confidential-containers/operator/config/samples/ccruntime/<CCRUNTIME_OVERLAY>?ref=<RELEASE_VERSION>
```

The corresponding `v0.8.0` release for `x86_64` is default, so run:

```
kubectl apply -k github.com/confidential-containers/operator/config/samples/ccruntime/default?ref=v0.8.0
```
And to deploy `v0.8.0` release for `s390x`, run:
```
kubectl apply -k github.com/confidential-containers/operator/config/samples/ccruntime/s390x?ref=v0.8.0
```
Wait until each pod has a status of running, and use the following command again:

```
kubectl get pods -n confidential-containers-system --watch
```

## Verify installation

Check the `RuntimeClasses` that got created.

```
kubectl get runtimeclass
```
Output:
```
NAME            HANDLER         AGE
kata            kata-qemu       6d
kata-clh        kata-clh        6d
kata-clh-tdx    kata-clh-tdx    6d
kata-qemu       kata-qemu       6d
kata-qemu-sev   kata-qemu-sev   6d
kata-qemu-snp   kata-qemu-snp   6d
kata-qemu-tdx   kata-qemu-tdx   6d
```

Details on each of the runtime classes:

- *kata* - standard kata runtime using the QEMU hypervisor including all CoCo building blocks for a non CC HW
- *kata-clh* - standard kata runtime using the cloud hypervisor including all CoCo building blocks for a non CC HW
- *kata-clh-tdx* - using the Cloud Hypervisor, with TD-Shim, and support for Intel TDX CC HW
- *kata-qemu* - same as kata
- *kata-qemu-tdx* - using QEMU, with TDVF, and support for Intel TDX CC HW, prepared for using Verdictd and EAA KBC.
- *kata-qemu-sev* - using QEMU, and support for AMD SEV HW
- *kata-qemu-snp* - using QEMU, and support for AMD SNP HW
