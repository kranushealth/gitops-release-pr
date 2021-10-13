# gitops-release-pr

This is a [reusable workflow](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) to do YAML update PR releases based on the GitOps pattern

### Examples:

The code below assumes the following setup:

- N repositories that get deployed via the gitops pattern
- through a manifest repository 
- ArgoCD + kustomize (but this would work with any other GitOps setup in a similar fashion!)

In this case in the manifest repo each app has a `/base/` folder and `/overlays/[environment-name]`. This base is patched with a `kustomization.yaml` in the respective overlay.

The `kustomization.yaml` of each overlay uses the following code to change the `Deployment` image tag (corresponds with the commmit id):

```
images:
- name: aws-repo-url
  newName: aws-repo-url
  newTag: commit-id
  
```

#####  So in order to do automatic PRs to the manifest repo when we merge each of the N repositories to the `dev` branch we do:

```yaml
name: CI

on:
  push:
    branches:
      - dev

jobs:
  release-dev-pr:
    name: DEV Release PR
    if: ${{ github.ref == 'refs/heads/dev' }}
    uses: kranushealth/gitops-release-pr/.github/workflows/pr.yaml@main
    with:
      pr-repository: some-org/manifests
      pr-target: main
      pr-branch: release/dev-app-${{ github.sha }}
      pr-title: "DEV - Release app 🚀"
      pr-body: "Image tag: ${{ github.sha }}"
      pr-commit-message: "release: dev app"
      pr-changes: |-
        yq eval -i '.images[0].newTag = "${{ github.sha }}"' ./applications/app/overlays/dev/kustomization.yaml
      pr-labels: |-
        release
        gitops
        DEV
    secrets:
      token: ${{ secrets.MY_GITHUB_PAT }}
```

You can also use the same logic to make PRs when someone pushes to the `main` branch of each respective repository to open release PRs.
