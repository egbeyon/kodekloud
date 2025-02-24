## Reconciliation loop - using timeout

```bash
kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"timeout.reconciliation":"300s"}}'
```

```bash
kubectl -n argocd rollout restart deploy argocd-repo-server
```

```bash
kubectl -n argocd describe pod argocd-repo-server | grep -i "ARGOCD_RECONCILIATION_TIMEOUT:" -B1
```

- Argocd will check for changes from the git repository every 5 minutes, the defaults is 3 minutes
  
- Alternatively, edit the configmap:
```bash
kubectl edit configmap argocd-cm -n argocd
```
