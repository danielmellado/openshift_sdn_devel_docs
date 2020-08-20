# How to test SDN OVN development on a Openshift cluster

Here you'll find resources for creating a DEV version of any OVN image and subsequently using that with `hack/run-locally.sh` to deploy your DEV image on a running OCP cluster. This document details the procedure for OVN, but the changes needed to make this work with SDN should be quite small.

You will need the following script to build a custom DEV image with any on-going work in either `openshift/ovn-kubernetes` or `ovn-org/ovn-kubernetes`:

`hack.sh`:

```
#!/bin/bash

set -ex

serial=$(uuidgen)

cd go-controller
make
cd ../local
cp -r ../go-controller/_output/go/bin/* ./

buildah build-using-dockerfile -t "quay.io/${QUAY_USER_NAME}/${QUAY_PUBLIC_PROJECT}:${serial}" -f Dockerfile.hack .
buildah push "quay.io/${QUAY_USER_NAME}/${QUAY_PUBLIC_PROJECT}:${serial}"
```

`Dockerfile.hack`:

```
FROM quay.io/openshift/origin-ovn-kubernetes:4.5

COPY ovnkube /usr/bin/
COPY ovn-k8s-cni-overlay /usr/libexec/cni/ovn-k8s-cni-overlay
```

Remember to swap out the `quay.io/openshift/origin-ovn-kubernetes:4.5` with the release version your DEV work corresponds to.

Once you've created a DEV version of your code and pushed that image to the image repository of your choosing, you can execute the following in the CNO to have them run on your OCP cluster:

`./hack/run-locally.sh -e`

That will write a file on your file system containing the environment variables the CNO uses to deploy the networking stack on the OCP cluster you've created.

Change the `OVN_IMAGE` or `SDN_IMAGE` to your DEV image name, then run:

`./hack/run-locally.sh`

The CNO should now be rolling out the DEV version on your cluster
