# GitHub ARC tests on OpenShift

This repository contains some Kubernetes resources which may be deployed onto an OpenShift cluster to test functionality of the GitHub Actions Runner Controller (ARC) when running on OpenShift.

To deploy, install the ARC CRDs and controller via Helm chart, as described on the [quickstart guide](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners-with-actions-runner-controller/quickstart-for-actions-runner-controller):

```
NAMESPACE="arc-systems"
helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
```

This will install the controller in namespace `arc-systems`.

Then, apply RBAC resources needed to deploy a runner set in the `arc-runners` namespace:

```
kubectl create namespace arc-runners # if not present
kubectl apply -n arc-runners -f rbac.yaml
```

Create a secret in the `arc-runners` namespace with a GitHub PAT:

```yaml
apiVersion: v1
kind: Secret
metadata: 
  name: runner-credentials
  namespace: arc-runners
data: 
  github_token: <your base64-encoded GitHub PAT>
type: Opaque
```

Finally, apply the resource of the type of runner set you wish to deploy:
- `runner-base.yaml`: basic runner set configuration, with no container mode set. Container-based jobs and actions will fail to execute.
- `runner-k8s.yaml`: Kubernetes-based runner set configuration, where container-based jobs and actions will be deployed in their own pod.
