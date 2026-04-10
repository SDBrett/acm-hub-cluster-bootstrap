# Overview

This repo is one of two repos which bootstrap the configuration of an OpenShift hub cluster. It is used to deploy OpenShift GitOps and two ArgoCD applications. One ArgoCD appliance deploys ACM and the other synchronizes ACM policies.

The second repo contains the ACM policies that performs the rest of the cluster configuration. The policies repo can be found here: https://github.com/SDBrett/acm-ocpv-policy-samples

End to end cluster deployment can be acheived when this repo is included as part of a pipeline or script.

The content is tested on the following products and versions:
- Red Hat OpenShift 4.20
- Red Hat Advanced Cluster Managerment 2.16
- OpenShift GitOps
  - AKA ArgoCD
- Kustomize 5.8

# Usage

## PreRequisites

* You have cloned this repo and the policies repo
* You have a base deployment of OpenShift
* The cluster has access to pull the required images
* You have cluster administrator permissions
* You have a system with kustomization


## Process

1. Update the content for your environment
   
    Most configuration that will be specific to an environment is located in the hub cluster overlay directories, though you may need to make additional changes to those listed below.

    1.1. Create hub cluster overlays

    Copy the directory `overlays/hub-template` for each hub cluster to be configured from this repo. These new directories will be refered to using the placeholder `HUB_CLUSTER_NAME`.
    
    1.2. Update patches

    The `overlays/HUB_CLUSTER_NAME/stage02` directory contains serveral patches for setting environmental configurations.

    Add values to `repoURL` and `targetRevision` in the `application-acm-policies-patches.yaml` and `application-hub-cluster-acm-patches.yaml` files.

    Add the policies repo URL to the `sourceRepos` list in the `appproject-acm-policies-patches.yaml` file.


2. Repo Secrets
   
   If credentials are required to access this repo and the policies then `Secrets` need to be created so OpenShift GitOps can authenticate.

   This can be done as part of this repo structure or as part of a cluster deployment pipeline / script.
   
   To do as part of this repo:

   In the `overlays/HUB_CLUSTER_NAME/stage01` directory create a file called `git-secrets.yaml`

   Example repo secrets can be found here: https://argo-cd.readthedocs.io/en/stable/operator-manual/argocd-repo-creds-yaml

   Uncomment the `git-secrets.yaml` line in the stage01 `kustomization.yaml` file.

   **Note:** The file name `git-secrets.yaml` is ignored by git.

3. Install OpenShift GitOps
   From the repo root run the command `kustomize build overlays/HUB_CLUSTER_NAME/stage01`.

   Continue to the next step after the OpenShift GitOps operator has been deployed and is running.

4. Deploy all the things
   From the repo root run the command `kustomize build overlays/HUB_CLUSTER_NAME/stage02`

   This will create the root intance of ArgoCD and the ArgoCD applications which will install ACM and synchronize the ACM policies.

   After ACM has been installed the policies will start configuring the cluster.

# Promoting Changes
The `git-diff.config` and `git-diff.sh` can be used to promote changes which is helpful if you have dedicated non-production and production hub clusters.

The process is configured around having using a production and non-production repo but it can be updated for branch based strategies.

The `git-diff.config` file is used to configure settings for the diff script such as the repo URL to compare the current repo against.

The script updates remotes for the source repo and compares it against your local copy. It will then create and apply a patch file. It does not perform any commits, it is assumed that you will manually compare the diff before commiting and creating a PR.

# Use of AI
AI is used for this project the primary uses are:
* Linting
* Testing kustomize build
* File renaming

Core content is written by hand.
