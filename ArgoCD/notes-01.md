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

## ArgoCD RBAC

1. Cluster-level RBAC
  - Create a role to create cluster, for a role assigned to 'jai':
```bash
kubectl -n argocd patch configmap argocd-rbac-cm \
--patch='{"data":{"policy.csv":"p, role:create-cluster, clusters, create, *, allow\ng, <jai>, role, role:create-cluster"}}'
```
  - Expected output: `configmap/argocd-rbac-cm patched`
    
  - This command can also be used for create-application role, edit the `cluster` for `create-app` 

  - Login as 'jai' and test the role:
```bash
argocd account can-i create clusters '*'
```
  - expected output: `yes`
  - but this role was not given the permission to delete a cluster


2. Project-level RBAC
  - Create a role with admin privileges for a project, for a role assigned to 'kia-admins':
```bash
kubectl -n argocd patch configmap argocd-rbac-cm \
--patch='{"data":{"policy.csv":"p, role:<kia-admins>, applications, *, <kia-project>/*, allow\ng, <ali>, role, role:<kia-admins>"}}'
```
  - `argocd account can-i sync applications <kia-project>/*`
  - expected output: `yes`
  - but this role cannot sync applications in other projects


## User management
1. Users can be added locally and through SSO
   
2. The local user can be given 2 capbilities
   - "apikey" and "login"
   - "apikey" allows creating JWT authentication for api-access
   - Login allows login access to use the UI
     
3. The available permissions to user roles on argocd are either admin or read-only. The admin would have to edit the configmap to upgrade the roles of subsequent users
   - Create account (the command below would update the configmap, by adding the account to the data field):
```bash
kubectl -n argocd patch configmap argocd-cm --patch='{"data":{"accounts.<jai>": "apiKey,login"}}'
```
  - Expected output `configmap/argocd-cm patched`
  - Updating a user's password
```bash
argocd account update-password --account <jai>
```

  - Assigning default read-only privilege to any user that is not mapped to a specific role:
```bash
kubectl -n argocd patch configmap argocd-rbac-cm --patch='{"data":{"policy.default": "role:readonly"}}'
```
