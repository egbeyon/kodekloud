## Create a serviceaccount for a dashboard monitoring application
```bash
kubectl create serciceaccount <name>
```
- A token is by generated during the creation of the serviceaccount (which can be viewed when the servicaccout is 'described'). The token is stored as a secret in the cluster.
- The token is used by the serviceaccount for authentication when accessing the kubernetes api 
