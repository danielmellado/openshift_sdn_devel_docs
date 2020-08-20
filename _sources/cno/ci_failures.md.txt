# Looking in to CI Failures

Diagnosing CI failures can be tricky. Here are some quick tips and hints for how to diagnose things.

## Failure to bootstrap

This sort of failure requires some debugging. Because the network is integral to the bootstrap process, you can't just assume that bootstrap failure is a flake. Retest-bot will constantly `/retest`, because it's programmed to do so on bootstrap failure.

These can be broadly categorized in to

### Cloud-provider issues

These are usually pretty obvious. You'll see some kind of Terraform issue and an error message complaining about the infrastructure provider. Retest and move on.

### CNO failures

A CNO failure will often be reflected in the ClusterOperator. It may also be printed to you in the CI failure. To diagnose this further, go to the test artifacts and view the cluster-network-operator pod logs.

### Networking issues

If your PR has broken networking, then the bootstrap failure will be more subtle. If it blocks pod creation, then basically every operator will be reporting failure. If pod creation works, but the network is down, then things will be a bit stranger. You'll see no worker nodes, because the machine-api-operator has failed. The console will fail to start. DNS will be broken, but the DNS operator probably won't detect this.

### Other flakes

It's a moving target. Flakes happen. But generally openshift-installer is reliable. You shouldn't dismiss bootstrap failures as a flake unless you're sure.

## Test failures

The test suite is, as a whole, flakier. However, there is a daily build-cop who's job it is to watch over tests and identify flakes, so they don't tend to last too long.

The challenge here is to differentiate flakes from bugs. The risk is that we `/retest` our way to a merge, even though we've introduced a bug. So, don't just `/retest` - identify the problem!

1. search over previous CI runs to see if the error message is common: [https://search.svc.ci.openshift.org/?search=your+error+message+here&maxAge=336h&context=2&type=junit](https://search.svc.ci.openshift.org/?search=your+error+message+here&maxAge=336h&context=2&type=junit)

2. If you see a lot of other recent failures, search Bugzilla to see if there is a bug for that error message

3. If you don't find a bug, ask the buildcop if it's a known flake.

4. If you don't see many hits, then you probably broke something. Go on over to [running the tests locally](run_local_e2e) and try to reproduce it yourself. Tail the logs of the networking components as you do. Good luck!
