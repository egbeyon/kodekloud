## Creating an ABAC Policy

- Create the abac-policy.jsonl file with the specified content:
```bash
cat <<EOF > /etc/kubernetes/abac/abac-policy.jsonl
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user": "system:serviceaccount:default:john", "namespace": "default", "resource": "pods", "apiGroup": "*" , "readonly": true}}
EOF
```

## Configuring the API Server to Use ABAC

- Edit the API server manifest file (usually located at /etc/kubernetes/manifests/kube-apiserver.yaml) and add the following flags under the command section:
```yaml
  # /etc/kubernetes/manifests/kube-apiserver.yaml

- --authorization-mode=Node,RBAC,ABAC
- --authorization-policy-file=/etc/kubernetes/abac/abac-policy.jsonl
```
- Add volume mounts to the API server pod spec:
```yaml
# /etc/kubernetes/manifests/kube-apiserver.yaml

volumeMounts:
# Other volume mounts...
- name: abac-policy
  mountPath: /etc/kubernetes/abac
  readOnly: true

volumes:
# Other volumes...
- name: abac-policy
  hostPath:
    path: /etc/kubernetes/abac
    type: DirectoryOrCreate
```
- The kube-apiserver will automatically restart sometime after the manifest file is updated. To check the logs, run cat /var/log/containers/kube-apiserver-*


## Create a Service Account and Retrieving Its Token
- Create policy, john-sa.yml:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: john
  namespace: default
secrets: 
  - name: john-secret
---
apiVersion: v1
kind: Secret
metadata:
  name: john-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: john
type: kubernetes.io/service-account-token
```
- Retrieve the service account token:
```bash
# Get the secret name
kubectl get serviceaccount john -o jsonpath='{.secrets[0].name}'

# Retrieve the token
kubectl get secret john-secret -o jsonpath='{.data.token}' | base64 --decode

# Save the token to a file
kubectl get secret john-secret -o jsonpath='{.data.token}' | base64 --decode > john-secret.txt
```


## Setting Up kubectl Credentials for the Service Account

- Set up new credentials:
```bash
export SA_TOKEN=$(cat john-secret.txt)
kubectl config set-credentials john --token=$SA_TOKEN
```
- Set up the new context:
```bash
kubectl config set-context john-context --cluster=kubernetes --namespace=default --user=john
kubectl config use-context john-context
```


## Testing the ABAC Configuration
```bash
kubectl get pods -n default
kubectl run test-pod --image=nginx
```
- Expected Outcome:
  - Listing pods should succeed.
  - Creating a pod should fail with a "Forbidden" error due to read-only access.

 
## Modify the ABAC Policy
- Switch back to kubernetes-admin@kubernetes context.
```bash
kubectl config use-context kubernetes-admin@kubernetes
```
- Update the /etc/kubernetes/abac/abac-policy.jsonl file:
```bash
{"apiVersion": "abac.authorization.kubernetes.io/v1beta1", "kind": "Policy", "spec": {"user": "system:serviceaccount:default:john", "namespace": "default", "resource": "pods", "readonly": false}}
```

- Restart the API server with new configuration:
- Test creating a pod from context
