# Gatekeeper playground


## Create cluster

```bash
K8S_VERSION=${k8sVersion:-v1.22.0@sha256:b8bda84bb3a190e6e028b1760d277454a72267a5454b57db34437c34a588d047}
CLUSTER_NAME=${CLUSTER_NAME:-gatekeeper}
cat <<EOF | kind create cluster --image kindest/node:${K8S_VERSION} --name ${CLUSTER_NAME} --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
featureGates:
nodes:
- role: control-plane
- role: worker
EOF
echo "Waiting on cluster to be ready..."
sleep 10
kubectl wait pod --timeout=-1s --for=condition=Ready -l '!job-name' -n kube-system > /dev/null
```

## Install Gatekeeper

Download gatekeeper-mutation
```bash
GATEKEEPER_VERSION=${GATEKEEPER_VERSION:-v3.5.2}
curl -fLO https://raw.githubusercontent.com/open-policy-agent/gatekeeper/$GATEKEEPER_VERSION/deploy/gatekeeper.yaml
```

```bash
kubectl apply -f gatekeeper.yaml
```

### Verify Gatekeeper installation

```bash
kubectl get pods -n gatekeeper-system
```


### Load demos


Clone the examples

```bash
git clone --depth 1 -b v3.5.2 git@github.com:open-policy-agent/gatekeeper.git
```

Try basic demo
```bash
pushd gatekeeper/demo/basic
./demo.sh
popd
```

Try basic agile bank demo
```bash
pushd gatekeeper/demo/agilebank/
./demo.sh
popd
```




## Clean up

Uninstall gatekeeper

```bash
kubectl delete -f gatekeeper.yaml
```

Delete cluster

```bash
kind delete cluster --name ${CLUSTER_NAME}
```