# Information about Deploying Bare Metal Clusters for Kubernetes, Open Shift and k OVN

## Deploying Kubernetes on bare metal.
### Standing up a simulated bare metal cluster using dev-scripts

dev-scripts is a collection of bash scripts and ansible code on github to deploy a kubernetes simulated bare metal cluster. The cluster is deployed on a bare metal server. the cluster is deployed
with k-OVN.
Each of the master and worker nodes are actually VMs each of which is running on the BM host.

[Dev Scripts on git hub.](https://github.com/openshift-metal3/dev-scripts)

[Google doc on using dev-scripts to deploy cluster.](https://docs.google.com/document/d/1F2Uh0lq-iAt0Qh9edmB3zyWnQBncYeNGpurq3YM31jc/edit#)

### Red Hat document about manually standing up Open Shift on bare metal.

### Deploying OpenShift on Bare Metal on User Provisioned Infrastructure (UPI) introduced in OCP 4.0

This method provisions a cluster including both bare metal nodes and VM nodes.
It is a Red Hat document on how to providion Open Shift 4 on UPI (User Provisioned Infrastructure)

[Deploying a UPI Environment for Openshift 4.1 on VMS and Bare Meta.](https://blog.openshift.com/deploying-a-upi-environment-for-openshift-4-1-on-vms-and-bare-metal/)
