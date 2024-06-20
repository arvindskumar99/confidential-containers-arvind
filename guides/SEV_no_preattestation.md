# SEV-ES guide with no pre-attestation

This guide covers platform-specific setup for SEV(-ES) and walks through
complete flows to launch COCO with no pre-attestation.

# Installation

You can enable Confidential Containers in an existing Kubernetes cluster using the Confidential Containers Operator.
When installation is finished, your cluster will have new runtime classes for different hardware platforms,
including a generic runtime for testing CoCo without confidential hardware support, a runtime using a remote hypervisor
that allows for cloud integration, a runtime for process-based isolation using SGX, as well as runtimes for TDX and SEV.

## Prerequisites


