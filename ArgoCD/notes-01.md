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

## Alter the health-check-status wrt confimap values
- `k get cm -n health-check moving-shapes-colors -o yaml`
```yaml
apiVersion: v1
data:
  CIRCLE_COLOR: pink
  OVAL_COLOR: lightgreen
  RECTANGLE_COLOR: blue
  SQUARE_COLOR: orange
  TRIANGLE_COLOR: white
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"CIRCLE_COLOR":"pink","OVAL_COLOR":"lightgreen","RECTANGLE_COLOR":"blue","SQUARE_COLOR":"orange","TRIANGLE_COLOR":"white"},"kind":"ConfigMap","metadata":{"annotations":{},"labels":{"app.kubernetes.io/instance":"health-check-app"},"name":"moving-shapes-colors","namespace":"health-check"}}
  creationTimestamp: "2025-02-25T16:38:29Z"
  labels:
    app.kubernetes.io/instance: health-check-app
  name: moving-shapes-colors
  namespace: health-check
  resourceVersion: "4349"
  uid: aa048bbb-c274-4e18-9ed5-e71d6a52e1d4
```

- Create custom `custom-cm.yaml` file to customize the configmap
```yaml
data:
  resource.customizations.health.ConfigMap: |
    hs = {}
    hs.status = "Healthy"
    if obj.data.TRIANGLE_COLOR == "white" then
      hs.status = "Degraded"
      hs.message = "Use any color other than White"
    end
    return hs
```

- Patch the configmap with the newly created yml file:
```bash
kubectl patch configmap argocd-cm -n argocd --patch-file custom-cm.yml
```
