# ArgoCD

GitOps deployment repo for [slack-bolt-app](https://github.com/arion849/slack-bolt-app). ArgoCD watches this repo and syncs the cluster to whatever it contains, so a deploy is just a commit here.

## How releases flow in

The app repo's GitHub Actions pipeline builds a versioned image, pushes it to JFrog Artifactory, then commits the new tag into `k8s/deployment.yaml` in this repo. ArgoCD picks up the change and rolls it out. Every line in this repo's commit history (`Update deployment to image tag 1.x`) is one release.

## Layout

| Path | What it is |
| --- | --- |
| `k8s/deployment.yaml` | The Deployment ArgoCD syncs: one replica of the bot, pulled from JFrog using `jfrog-registry-secret`, with Slack credentials read from the `slack-env` secret |
| `k8s/service.yaml` | NodePort Service (port 80 → container 3000, node port 30080) |
| `helm/slack-bolt-app/` | A Helm chart scaffold for the same app (not yet wired to the real image or values) |

## Cluster prerequisites

```bash
kubectl create secret docker-registry jfrog-registry-secret \
  --docker-server=<registry>.jfrog.io --docker-username=<user> --docker-password=<token>
kubectl create secret generic slack-env \
  --from-literal=SLACK_BOT_TOKEN=xoxb-... --from-literal=SLACK_SIGNING_SECRET=...
```

Then point an ArgoCD Application at this repo with path `k8s`.
