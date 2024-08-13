# Kubernetes GHA For CNCF

This goes over the generic setup of a Kubernetes cluster that's sole purpose is to be a scale-down GitHub action runner server.

Long-term, this should be the primary method for projects to consume our provider-donated resources. 

Also, for every type of size offering, we will need to create a new namespace/runnerScaleSet with the appropriate values. 

## Setup

1. Install GitHub's Action Runner Controller (ARC):
   ```
   NAMESPACE="arc-systems"
   helm install arc \
    --namespace "${NAMESPACE}" \
    --create-namespace \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set-controller
   ```

2. Snag a PAT from GitHub. This.... begrudgingly... needs to be a user PAT. Sorry.

3. Install a GH ARC RunnerSet:
   ```
   INSTALLATION_NAME="provider-runner-size"
   NAMESPACE="provider-runner-size"
   GITHUB_CONFIG_URL="https://github.com/enterprises/cncf"
   GITHUB_PAT="<PAT GOES HERE>"
   helm install "${INSTALLATION_NAME}" \
    --namespace "${NAMESPACE}" \
    --set githubConfigUrl="${GITHUB_CONFIG_URL}" \
    --set githubConfigSecret.github_token="${GITHUB_PAT}" \
    -f runner-values.yaml \
    oci://ghcr.io/actions/actions-runner-controller-charts/gha-runner-scale-set
   ```

4. Once everything has synced and started, the Runner type will be available in the CNCF GHE.

## Runner

By default, the GitHub runner image does not have certain dependencies (such as `git`). There is an ongoing discussion around this [here](#). For now, we need to build-and-run our own [Image](./Dockerfile)
