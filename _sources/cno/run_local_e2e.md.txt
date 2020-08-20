# HOWTO run the e2e test suite locally

Running the OpenShift / Kubernetes e2e test suite against a "local" cluster is easy. All you need to do is:

1) Check out https://github.com/openshift/origin/
2) `make WHAT=cmd/openshift-tests`
3) Ensure the `KUBECONFIG` environment variable is set. Then, you can run a test or a suite

### Running a single test
use the `run-test` subcommand. Pass it the full test name, with all the brackets. e.g.

```bash
KUBE_SSH_USER=core KUBE_SSH_KEY_PATH=~/.ssh/WHATEVER \
    _output/local/bin/linux/amd64/openshift-tests run-test \
    "[Conformance][Area:Networking][Feature:Router] The HAProxy router should expose prometheus metrics for a route [Suite:openshift/conformance/parallel/minimal]"
````

### GCP credentials

You might need to extract credentials if you're running on GCP. This should do the trick:

```bash
oc -n openshift-machine-api get secret gcp-cloud-credentials -o jsonpath='{.data.service_account\.json}' | base64 -d > gcp-key.json
export GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp-key.json
```

### Running a test suite
use the `run` subcommand
```bash
KUBE_SSH_USER=core KUBE_SSH_KEY_PATH=~/.ssh/WHATEVER \
    _output/local/bin/linux/amd64/openshift-tests\
    run  kubernetes/conformance
```

You can get the list of test-suites with `--help`. If you're doing network hacking, then the most useful suites are `kubernetes/conformance` and `openshift/conformance`.
