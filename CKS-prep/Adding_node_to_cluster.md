## Using Bootstrap token

- Create a Kubernetes Secret in the kube-system namespace to define a bootstrap token <bootstrap-token.yaml>:
```bash
apiVersion: v1
kind: Secret
metadata:
  name: bootstrap-token-07401b
  namespace: kube-system
type: bootstrap.kubernetes.io/token
stringData:
  description: "Bootstrap token for adding new nodes securely."
  token-id: 07401b
  token-secret: f395accd246ae52d
  usage-bootstrap-authentication: "true"
  usage-bootstrap-signing: "true"
  auth-extra-groups: system:bootstrappers:kubeadm:default-node-token
```
  - `kubectl apply -f bootstrap-token.yaml`

- Run the following command on the control plane node
```bash
kubeadm token create [random-token-id].[random-secret] --dry-run --print-join-command --ttl 2h
```
  - Store the CA certificate hash at `~/ca-cert-hash` for future reference

- On the new worker node named node01, join the cluster securely using the bootstrap token <bootstrap-token-07401b>
  - ssh to the node: `ssh <node_name>`
  - ensure kubeadm, kubelet and kubectl are installed on the new node. On the the node, run:
```bash
# Get the join command from the control plane node
# Obtain the <CONTROL_PLANE_ENDPOINT> vak `kubectl get node -o wide` and <HASH> from the previous step
kubeadm join <CONTROL_PLANE_ENDPOINT>:6443 --token 07401b.f395accd246ae52d --discovery-token-ca-cert-hash sha256:<HASH>
```
  - run `kubectl get nodes` on the controlplane to confirm the addition of the node to the cluster
