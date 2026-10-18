# Reddit Clone: GitOps Repository

Kubernetes manifests and the Jenkins CD pipeline for the [Reddit Clone CI/CD project](https://github.com/KaveenAlahapperuma/A-Reddit-Clone). **ArgoCD** watches this repo and keeps the AWS EKS cluster in sync with it.

## Files

| File | Purpose |
|---|---|
| `deployment.yaml` | Deployment running the `kaveend4/reddit-clone-app` image on port 3000 |
| `service.yaml` | `LoadBalancer` Service exposing the app on port 3000 |
| `Jenkinsfile` | `Reddit-Clone-CD` pipeline that updates the image tag |

## How It Works

```
Reddit-Clone-CI (Jenkins) ──IMAGE_TAG──► Reddit-Clone-CD (Jenkins)
                                              │
                                              │ sed: update image tag in deployment.yaml
                                              │ git commit + push
                                              ▼
                                       this repository
                                              │
                                              ▼
                                         ArgoCD sync
                                              │
                                              ▼
                                       AWS EKS cluster
```

1. The CI pipeline builds a new image (`1.0.0-<BUILD_NUMBER>`) and triggers this CD job with the tag.
2. The CD job checks out this repo and updates the image line in `deployment.yaml`.
3. It commits and pushes the change to `main`.
4. ArgoCD (auto-sync enabled) detects the new commit and rolls out the new version to EKS. Each rollout creates a new ReplicaSet, so every revision shows up in ArgoCD's history and can be rolled back.

Screenshots of the ArgoCD sync and resource tree are in the [main project README](https://github.com/KaveenAlahapperuma/A-Reddit-Clone#argocd-application-synced-and-healthy).

## Lesson Learned: Keep the Tag Update Narrow

An early version of the `sed` command in the `Jenkinsfile` was too broad and also changed the Deployment's selector and labels. Kubernetes doesn't allow `spec.selector` to change on an existing Deployment, so the rollout failed with an immutable-field error. The fix was to match only the container image line:

```bash
sed -i 's|image: kaveend4/${APP_NAME}:.*|image: kaveend4/${APP_NAME}:${IMAGE_TAG}|' deployment.yaml
```

## Why a Separate Repo?

- Keeps deployment config apart from application code
- Git history shows every version that was deployed, and when
- Rolling back is just reverting a commit
- Jenkins never needs direct access to the cluster; ArgoCD pulls the changes

Main project README: [A-Reddit-Clone](https://github.com/KaveenAlahapperuma/A-Reddit-Clone)
