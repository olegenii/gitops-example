# GitOps-example repo
## Description
This is a training repo for exploring newest tools and technologies as GitOps, ControlPlanes and etc.

>>Note: All scripts were created for Mac M1 CPU!

Repo consist of one control-cluster `kind-control-plane` definition with `Crossplane` running in `kind` locally and a target small workload cluster `d1-blue` running in `Azure AKS` created by `Crossplane`, which was mentioned earlier. Both clusters configuration described with `FluxCD` manifests.

Kind cluster going to be bootstrapped with bashscript. Then another script install flux operator in kind control cluster. Finally flux will do other steps itself.

All secrets store inside `.env` file.

## Installation steps
1. Install toolset on your machine:
    - `kubectl`*
    - `helm`*
    - `kind`* - local k8s cluster
    - `gh` - work with github
    - `ngrok` - exposing local container outside for exposing.
    * - _mandatory tools_
2. Create/fork a repo where flux will store state.
3. Create a [GitHub App for authentication to flux repo](https://fluxcd.io/blog/2025/04/flux-operator-github-app-bootstrap/#bootstrap-using-flux-operator-and-helm).
4. Save pem cert from GHApp into `ghapp.pem` file. Install GHApp into flux repo with at least write code permission.
5. Execute `start.sh` - it creates kind cluster and bootstrap flux into it.

Finish with `stop.sh` - it destroy kind cluster. 
>> Note: Check Azure resource after destroying kind control cluster!

>> Note: I prefer `zsh` as working shell with some plugins and autocompletion to speed up Ops tasks.
