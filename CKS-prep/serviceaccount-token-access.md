## Create Service Account to access API server

- Create a new Service Account named my-service-account in the default namespace.
  - Create a secret named my-service-account-token to store the token.
  - Associate the secret with the Service Account.
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  namespace: default
secrets:
  - name: my-service-account-token
---
apiVersion: v1
kind: Secret
metadata:
  name: my-service-account-token
  namespace: default
  annotations:
    kubernetes.io/service-account.name: "my-service-account"
type: kubernetes.io/service-account-token
```
```bash
kubectl apply -f serviceaccount.yaml
```

- Get the name of the secret associated with the Service Account, then decode the token:
```bash
SECRET_NAME=$(kubectl get serviceaccount <my-service-account> -o jsonpath='{.secrets[0].name}')
kubectl get secret $SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode
```

- If not existing, create a Role and RoleBinding to grant the Service Account specific permissions to access pods in the default namespace:
```yaml
# Role to read pods
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
# RoleBinding to bind the role to the Service Account
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-service-account
  namespace: default
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f role.yaml
```
- Verify the setup:
```bash
kubectl auth can-i list pods --as=system:serviceaccount:default:<my-service-account>
```

- Use the retrieved token to Access the Kubernetes API server:
```bash
# get api-server endpoint
APISERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')

# get the CA certificate
CACERT=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.certificate-authority}')
kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 --decode > ca.crt

# retrieve the token
SECRET_NAME=$(kubectl get serviceaccount my-service-account -o jsonpath='{.secrets[0].name}')
TOKEN=$(kubectl get secret $SECRET_NAME -o jsonpath='{.data.token}' | base64 --decode)

# access the api-server
curl --cacert ca.crt -H "Authorization: Bearer $TOKEN" "$APISERVER/api/v1/namespaces/default/pods"
```
