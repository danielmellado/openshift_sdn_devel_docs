# Using cluster bot to grab new debug information from CI testing

When issues arise in the CI testing it is difficult to get additional debug information from the system because it is not easy to do so basic debug operations.

## Cluster-bot
cluster-bot is a slack app that can create interactive clusters and run CI tests. At the time of writing this this is the help text from cluster-bot
```
help - help
launch image_or_version_or_pr options - Launch an OpenShift cluster using a known image, version, or PR. You may omit both arguments. Use nightly for the latest OCP build, ci for the the latest CI build, provide a version directly from any listed on https://openshift-release.svc.ci.openshift.org, a stream name (4.1.0-0.ci, 4.1.0-0.nightly, etc), a major/minor X.Y to load the latest stable version for that version (4.1), <org>/<repo>#<pr> to launch from a PR, or an image for the first argument. Options is a comma-delimited list of variations including platform (aws, gcp, azure, vsphere, metal) and variant (ovn, proxy, compact, fips, mirror, shared-vpc, large, xlarge, ipv6, preserve-bootstrap, test).
Example: launch openshift/origin#49563 gcp
lookup image_or_version_or_pr - Get info about a version.
list - See who is hogging all the clusters.
refresh - If the cluster is currently marked as failed, retry fetching its credentials in case of an error.
done - Terminate the running cluster
auth - Send the credentials for the cluster you most recently requested
test upgrade from to options - Run the upgrade tests between two release images. The arguments may be a pull spec of a release image or tags from https://openshift-release.svc.ci.openshift.org. You may change the upgrade test by passing test=NAME in options with one of e2e-upgrade, e2e-upgrade-all, e2e-upgrade-partial, e2e-upgrade-rollback
test name image_or_version_or_pr options - Run the requested test suite from an image or release or built PRs. Supported test suites are e2e, e2e-serial, e2e-all, e2e-disruptive, e2e-disruptive-all, e2e-builds, e2e-image-ecosystem, e2e-image-registry, e2e-network-stress. The from argument may be a pull spec of a release image or tags from https://openshift-release.svc.ci.openshift.org.
build from - Create a new release image from one or more pull requests. The successful build location will be sent to you when it completes and then preserved for 12 hours.
version - Report the version of the bot
```

for the purposes of this information we are most interested in
```
test name image_or_version_or_pr options - Run the requested test suite from an image or release or built PRs. Supported test suites are e2e, e2e-serial, e2e-all, e2e-disruptive, e2e-disruptive-all, e2e-builds, e2e-image-ecosystem, e2e-image-registry, e2e-network-stress. The from argument may be a pull spec of a release image or tags from https://openshift-release.svc.ci.openshift.org.
```

this allows the user to run a series of different CI tests from a pull request. Editing the CNO, sdn, or ovn-kubernetes to print out additional debug information posting a PR to the respective branch and launching the test from cluster-bot allows us to target periodic jobs that are not part of our standard testing

`test e2e-network-stress openshift/ovn-kubernetes#192 ovn` is an example it is running the e2e-network-stress using the ovn-kubernetes image created in [https://github.com/openshift/ovn-kubernetes/pull/192](https://github.com/openshift/ovn-kubernetes/pull/192) when the test completes cluster-bot sends a slack message and link to the logs
