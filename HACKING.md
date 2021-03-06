# Cluster Ingress Operator Hacking


## Local development

It's possible (and useful) to develop the operator locally targeting a remote cluster.

### Prerequisites

* An OpenShift cluster with at least a master, infra, and compute node. **Note**: in most cases, the master and infra nodes should *not be colocated* for testing the ingress operator. This is because Kubernetes will reject master nodes as LoadBalancer Service endpoints.
* An admin-scoped `KUBECONFIG` for the cluster.
* The [operator-sdk](https://github.com/operator-framework/operator-sdk).

#### GCP test clusters

One reliable and hands-free way to create a suitable test cluster and `KUBECONFIG` in GCP is to use the [openshift/release](https://github.com/openshift/release/tree/master/cluster/test-deploy) tooling. The default `gcp-dev` profile will produce a cluster compatible with Service Load Balancers.

### Building

To build the operator during development, use the standard Go toolchain:

```
$ go build ./...
```

### Container image

To build the operator docker image:

##### Using docker

```
docker build -t openshift/cluster-ingress-operator:latest .
```

##### Using buildah

```
sudo buildah bud -t openshift/cluster-ingress-operator:latest .
```

```
sudo buildah push openshift/cluster-ingress-operator:latest docker-daemon:openshift/cluster-ingress-operator:latest
```


### Running

To run the operator, first deploy the custom resource definitions:

```
$ oc create -f manifests/02_crd.yaml
```

#### Running with Operator SDK

Use the operator-sdk to launch the operator:

```
$ operator-sdk up local namespace default --kubeconfig=$KUBECONFIG
```

If you're using the `openshift/release` tooling, `KUBECONFIG` will be something like `$RELEASE_REPO/cluster/test-deploy/gcp-dev/admin.kubeconfig`.

#### Running a Containerized Operator

```
oc create -f manifests/04_cluster-ingress-operator.yaml
```

*In order to run as a container you will need to upload the Cluster Ingress Operator image to your registry*


## Tests

### Integration tests

Integration tests are still very immature. To run them, start with an OpenShift 4.0 cluster and then run the following,
substituting for your own details where appropriate. This assumes `KUBECONFIG` is set.

```
# 1. Uninstall any existing cluster-version-operator and cluster-ingress-operator.
$ hack/uninstall.sh

# 2. Build and push a new cluster-ingress-operator image.
$ REPO=docker.io/username/origin-cluster-ingress-operator make release-local

# 3. Run `oc apply` as instructed to install the locally-built operator.

# 4. Run integration tests against the cluster.
$ CLUSTER_NAME=your-cluster-name make test-integration
```

**Important**: Don't run these tests in a cluster where data loss is a concern.
