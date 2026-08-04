# notes-gitops — what is actually running

This repo holds **what to run**. Argo CD watches it and continuously makes your cluster match it.

> **This repo IS the cluster.** If `apps/notes-api/deployment.yaml` says `replicas: 3`, three Pods
> run. To change the cluster you do **not** run `kubectl` — you change a file here and push.

```
apps/notes-api/
  deployment.yaml   ← the image tag lives here; CI rewrites it on every push
  service.yaml
  ingress.yaml
argocd/
  application.yaml  ← apply this ONCE to tell Argo CD to watch this repo
```

## How a deploy happens

You never edit the image tag by hand. Every push to `main` in **`notes-application`** builds a new
image and **commits the new tag into this repo**. That commit *is* the deploy — so this repo's git
log is your **deploy history**:

```bash
git log --oneline apps/notes-api/deployment.yaml
#   deploy notes-api 9f2c1a...   ← each line is a release
```

## Two things you should try

**Rollback** — undo a release by undoing its commit:
```bash
git revert --no-edit HEAD && git push
# Argo CD syncs the cluster back to the previous image
```

**Drift** — change the cluster by hand and watch it get undone:
```bash
kubectl -n notes-api scale deploy notes-api --replicas=7
sleep 15
kubectl -n notes-api get deploy notes-api     # back to what THIS repo says
```
That's `selfHeal`. Your manual change is gone — **to change the cluster, change Git.**

## Setup

1. Leave `apps/notes-api/deployment.yaml` alone — it ships pointing at the course's public image so
   your first sync works immediately. CI rewrites that line on your first push.
2. `argocd/application.yaml` → set `repoURL` to your own `notes-gitops` repo.
3. Apply it once: `kubectl apply -f argocd/application.yaml`
