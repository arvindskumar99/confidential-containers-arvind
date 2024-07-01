# SNP guide

CoCo supports many Trusted Execution Environments (TEE) including Intel TDX, AMD SEV, AMD SNP, and many more. This guide covers platform-specific setup for an instance of CoCo using SNP.

### Platform Setup


This guide assumes that you've already installed the [snp.sh](https://github.com/amd/sev-utils/tree/coco-202402240000) script, a tool that builds the required patched versions of qemu ovmf, and a linux kernel, tools you will need to create your COCO instance. 

Ensure that you've followed the steps to prepare the host as stated [here](https://github.com/AMDESE/AMDSEV/tree/snp-latest). These instructions will help you setup SNP on your host level, which is required in order to create the SNP CoCo.

 For SNP, all that is required is the operator and the runtime class. The corresponding runtime class for launching with SNP is `kata-qemu-snp`. Now that you've installed and deployed the operator, you can go on to creating a kubernetes service yaml file. The yaml file is a config file that determines and tells how your kubernetes pods should run and interact with other objects. 

### Launch the Pod 
```
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
  annotations:
    io.containerd.cri.runtime-handler: kata-qemu-snp
    io.katacontainers.config.pre_attestation.enabled: "false"
spec:
  containers:
  - image: bitnami/nginx:1.22.0
    name: nginx
  dnsPolicy: ClusterFirst
  runtimeClassName: kata-qemu-snp
  ```

  Create a service yaml similar to this nginx.yaml file. Notice how the runtime class' name is set to kata-qemu-snp.

  Start the service:

  ```
  kubectl apply -f nginx.yaml
  ```

Check the pod for errors using:

```
pod_name=$(kubectl get pod -o wide | grep encrypted-image-tests | awk '{print $1;}')
kubectl describe pod ${pod_name}
```

If there are no errors, then the CoCo container has been successfully created.