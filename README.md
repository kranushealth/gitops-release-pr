# gitops-release-pr

This is a [reusable workflow](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows) to do YAML update PR releases

### Examples:

#####  Only dev:

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
