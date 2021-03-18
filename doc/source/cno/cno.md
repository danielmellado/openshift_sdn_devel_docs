Running a local cluster network operator for plugin development
===============================================================

## Creating a Custom Image
### Create an image repository
You need somewhere to store your custom image that the AWS cluster instance can pull it from. The easiest options are [Docker Hub](https://hub.docker.com/) or [Quay.io](https://quay.io/signin/). Create an account. Now create a repository called "ovn-kubernetes".

The typical development cni images are for ovn-kubernetes and sdn. ovn upstream is at git@github.com:ovn-org/ovn-kubernetes.git. Downstream is git@github.com:openshift/ovn-kubernetes.git The installer includes images built in downstream. sdn is git@github.com:openshift/sdn

## Creating a Custom ovn-kubernetes Image
### Clone ovn-kubernetes
1. `git clone git@github.com:ovn-org/ovn-kubernetes.git`
2. make your changes to the source tree
3. `cd ovn-kubernetes/go-controller`
4. `make`

### Build and push the custom ovn-kubernetes image
Assuming you are working with the [upstream ovn-kubernetes git repo](https://github.com/ovn-org/ovn-kubernetes) and using Docker Hub, you can build an image from your ovn-kubernetes changes using the following steps:
1. `cd dist/images/`
2. `make fedora`
3. `docker tag ovn-kube-f:latest docker.io/(docker hub username)/ovn-kubernetes:latest`
4. `docker push docker.io/(docker hub username)/ovn-kubernetes:latest`

### Using Downstream/OpenShift ovn-kubernetes with podman
1. Make your changes to your local openshift/ovn-kubernetes repo
2. Push your changes to a new PR
3. Click the 'details' link on the prow/images CI job in your PR
4. Refresh until the build log appears, and look for the line `Tagged shared images from ocp/4.7:${component}, images will be pullable from registry.buildXX.ci.openshift.org/XXXXXXX/stable:${component}`
5. Login into build cluster console: e.g. https://console.buildXX.ci.openshift.org/ where 'XX' is the cluster number.
6. Click on the question mark on the upper-left portion of the screen and select 'Commmand Line Tools' and 'Copy Login Command'to get the `oc login` command and token.
7. Run the command you got from the previous step: e.g. `oc login --token=<token> --server=https://api.build02.gcp.ci.openshift.org:6443`
8. Run `oc registry login --to /path/to/secret`
9. Run `sudo podman pull --authfile /path/to/secret registry.buildXX.ci.openshift.org/XXXXXXX/stable:ovn-kubernetes` substituting the right `ci-op-xxxx` namespace from step 5.
10. Push the image to dockerhub with `sudo podman push`

### Using Downstream/OpenShift ovn-kubernetes with docker
1. Make your changes to your local openshift/ovn-kubernetes repo
2. Push your changes to a new PR
3. Click the 'details' link on the prow/images CI job in your PR
4. Refresh until the build log appears, and look for the line `Tagged shared images from ocp/4.7:${component}, images will be pullable from registry.buildXX.ci.openshift.org/XXXXXXX/stable:${component}`
5. Login into build cluster console: e.g. https://console.buildXX.ci.openshift.org/ where 'XX' is the cluster number.
6. Click on the question mark on the upper-left portion of the screen and select 'Commmand Line Tools' and 'Copy Login Command'to get the `oc login` command and token.
7. Run the command you got from the previous step: e.g. `oc login --token=<token> --server=https://api.build02.gcp.ci.openshift.org:6443`
8. Run `oc registry login`
9. Run `docker login --username $(oc whoami) --password $(oc whoami -t) registry.buildXX.ci.openshift.org` where 'XX' is the build cluster number
10. Run `docker pull registry.buildXX.ci.openshift.org/XXXXXXX/stable:ovn-kubernetes` substituting the right `ci-op-xxxx` namespace from step 4 and the build cluster number.
11. Push the image to dockerhub with `docker push`

## Download the OpenShift Installer
The installer handles cluster creation in AWS, gcp, Azure and more.
1. Go to [https://openshift-release.svc.ci.openshift.org/](https://openshift-release.svc.ci.openshift.org/) and find an installer in the "4.6" stream that is shown in green (eg, has passed CI).

The selected installer includes installer and client images for Linux and Mac. Always grab both an installer and the related client tools. You will frequently need to upgrade the installer and tools so put them in a convenient directory and add the directory to your $PATH (in ~/.baserc).

The nightly installers are built with the official OCP images and the ci builds are built with the images that are used in ci testing. Aside from the cluster-network-operator which we will be building and testing, all the images are part of the chosen installer. So if you need a specific image you need an installer that has it.

2. Click the "Download the installer" link at the top of the page
3. Wait a while
4. Click the link for the "openshift-install-linux" tarball (eg "openshift-install-linux-4.2.0-0.ci-2019-06-25-135202.tar.gz") to download it
5. Extract the tarball somewhere for later, like /tmp

## Get your Pull Secrets
Pull Secrets are specific to your user and allow your cluster to download the container images for all the OpenShift components. There are two pull secrets to get: the internal-only OpenShift CI ones (used by some installer bootstrap images) and the general OpenShift Developer secrets. The pull-secrets do change periodically. It's best to make sure you have the latest pull-secrets before attempting to spawn the cluster if it's been a few days since you got your pull-secrets updated.

### Get the Generic OpenShift pull secrets
The generic pull secrets are the same for all clusters. The "AWS" choice is a good as any other.
1. go to the [OpenShift portal secrets](https://cloud.openshift.com/clusters/install#pull-secret) page
2. Click the big "AWS" box
3. Click the "Installer Provisioned Infrastructure" box
4. Click the "Download Pull Secret" box and save the pull secret to `/tmp/pull-secret`

### Get the OpenShift CI pull secret
1. go to [https://api.ci.openshift.org/console/](https://api.ci.openshift.org/console/)
2. Click the (?) in the upper right
3. click on "Command Line Tools" in the menu that drops down
4. Click the Clipboard icon at the end of the "oc login https://..." box
5. Paste that command from the clipboard into a terminal and run it
5. Run `oc registry login --to=/tmp/pull-secret` to dump the pull secret to the same file as before

Your CI pull secret is also now in `/tmp/pull-secret` combined with the generic OpenShift pull secrets that we'll feed to the installer.

### Using the combined pull secret
Once we've combined the pull secret file you can pass them to the installer when it asks for them.
simply by copying the output of `jq -c < /tmp/pull-secret`

## Clone and build the Cluster Network Operator
1. `git clone git@github.com:openshift/cluster-network-operator.git`
2. `cd cluster-network-operator`
3. `hack/build-go.sh`

Whenever you make changes in the Cluster Network Operator you must `hack/build-go.sh` to include the changes in testing.

## Get an AWS account
https://mojo.redhat.com/docs/DOC-1081313#jive_content_id_Amazon_AWS

## Get a GCE account
https://mojo.redhat.com/docs/DOC-1081313#jive_content_id_GCE
The Service Account is required the first time you create a GCP cluster. It is cached for subsequent use.

## Run hack/run-locally.sh to start a cluster with your custom image
The first time you run the installer it will ask a series of questions. One of the questions is the cluster host. When it is done there will be an 'install-config.yaml.bak.XXXXXX' file in the cluster temporary directory that you give to hack/run-locally.sh. You can copy this 'install-config.yaml.bak.XXXXX' file and pass it to hack/run-locally.sh to save steps next time.

The installer puts its work files in a supplied directory (which it will create if not present). Otherwise, it puts them in the current directory.

The default cni plugin is sdn. The -n option can be used to select ovn.

3. Run `hack/run-locally.sh -c (cluster temp dir) -n ovn -m docker.io/(docker hub username)/ovn-kubernetes:latest` and substitute as necessary.
4. hack/run-locally.sh runs the openshift-installer for you, so now you get to answer some questions
5. `SSH Public Key` - this allows you to SSH to the bootstrap node for debugging; pick one.
6. `Platform` - pick AWS or gcp
7. `Region` - pick something close to you (the default selection tends to get oversubscribed)
8. `Base Domain` - pick devcluster.openshift.com
9. `Cluster Name` - pick something unique like "dcbw-ovntest". (It should start with your redhat username).
10. `Pull Secret` - Paste the contents of `/tmp/combined-pull-secrets` that we created earlier
11. Your install-config.yaml.bak.XXXXX file will now be in your (cluster temp dir). Copy this file somewhere for future use.

It is convenient to put the kubeconfig into the KUBECONFIG environment variable. This eliminates the need for the --config option. `export KUBECONFIG=/(cluster temp dir)/auth/kubeconfig`

## Poking around your cluster
1. You can `tail -f /(cluster temp dir)/.openshift-install.log` to watch progress. After it says "
2. List pods: `oc get pods --all-namespaces` and look for ovn-kubernetes related pods. You should see them running.
3. Get logs from ovn-kubernetes: `/path/to/oc --config /(cluster temp dir)/auth/kubeconfig logs -n openshift-ovn-kubernetes (ovn-kubernetes pod name)` and it will yell at you to pick a container. Pick one and repeat the previous command but add `-c (container name)` to the end to get the logs.

AWS cluster:
You can SSH to the bootstrap node if things don't seem to be coming up after a while. Look in `/(cluster temp dir)/terraform.tfstate` for the `aws_instance` type with the name "bootstrap". About 20 lines below you'll see "public_ip"; copy that IP and `ssh -i /path/to/openshift-dev.pem core@(public IP)`. You get `openshift-dev.pem` from the [shared secrets](https://github.com/openshift/shared-secrets) repository. Then `journalctl -b -f -u bootkube.service` and take a look at the errors.

GCP cluster:
You can SSH to the bootstrap node if things don't seem to be coming up after a while. Look in `/(cluster temp dir)/terraform.tfstate` for the first `"access_config":` a few lines below is `"nat_ip": "35.221.35.206" which is the public IP address of the bootstrap node. `ssh -i ~/.ssh/\(rsa key)/ core@(public IP)`

On the bootstrap node, `oc get nodes -o wide` shows the internal IP of the nodes. Copy the rsa key you selected in the cluster create into `~/.ssh`
From the bootstrap node you can ssh into the master nodes `ssh -i \(rsa key)\ core@\(internal node ip)\`

## Destroying your cluster
1. Ctrl+C hack/run-locally.sh
2. Run `/path/to/openshift-install --dir (cluster temp dir) destroy cluster`
3. Wait a long time

## Subsequent cluster starts
Since you cached the install-config you can save yourself a lot of time. Now all you need to do is:
1. Run `PATH=/path/to/oc:$PATH hack/run-locally.sh -c (cluster temp dir) -i /path/to/openshift-install -n ovn -m docker.io/(docker hub username)/ovn-kubernetes:latest -f /path/to/install-config.yaml` and substitute as necessary.
