# LodeStar - Deployment

This repository both bootstraps and manages the deployment lifecycle of LodeStar across multiple versions. It contains a manifest which controls the release version of all individual components of LodeStar. It is meant to be used with Argo CD.

## Bootstrapping

To create an instance of LodeStar in a cluster of your choice, ensure you're on the branch/commit-id of your choice and you have working oc/kubectl session towards the target cluster, then  apply the `bootstrap` Kustomize using the following command:

```sh
kustomize build bootstrap/ | oc apply -f -
```

NOTE: Running `kustomize build bootstrap/` alone will print manifests that can be reviewed before actually applying it to the cluster.

## Lifecycle Management

LodeStar is not a single application, but a collection of components (or services) that all add up to a working system. Those services might be versioned separately from one another. Therefore, a "deployment of LodeStar" is really a manifest of the versions of its constituent components that are known to work well with each other.

Kustomize patches for respective components live in `bootstrap/patches/` directory.

NOTE: Depending on the component, version can be stored in either `targetRevision` variable, or `targetRevision` and `imageTag` variables.

Automatic syncing, pruning, and self healing is enabled for this project - so committing a change to the above manifest and tagging it appropriately as `deploy-staging` or `deploy-production` will cause an Argo CD instance bootstrapped with that tag to deploy those versions of the applications.

## Releases

Creating releases is automated through the use of GitHub Actions Workflows. To create a release, you can just create an issue in this repository named as the release you'd like to create  using the "Creating a release" template, and use the below triggers in an issue comment to begin:

`/release` - This trigger along with the appropriate parameters can be used to create a release branch that will then be used to promote the appropriate changes into a new environment branch. The following parameters can be passed to it:

| Variable | Description | Required |
|:---------|:------------|:---------|
|frontend|The version to use for the frontend application|no|
|backend|The version to use for the backend application|no|
|gitapi|The version to use for the Git API|no|
|dispatcher|The version to use for the Resource Dispatcher|no|
|activity|The version to use for activity application|no|
|artifacts|The version to use for artifacts application|no|
|participants|The version to use for participants application|no|
|agnosticv|The version to use for AgnosticV|no|
|anarchy|The version to use for Anarchy|no|
|poolboy|The version to use for Poolboy|no|

`/promote` - This trigger along with the appropriate parameters can be used to promote the changes contained in a release branch into a specified environment branch that ArgoCD is watching. These promotions are auto-merged, so you should be sure that you are ready for a promotion when you run this command. It does not accept any parameters.

`/cancel` - This trigger cancels the rollout/closes the release issue and signals that a new issue should be created if a release is still desired.
