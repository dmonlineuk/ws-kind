# ws-kind

Plan is to have as small an impact as possible on my Arch install. I have podman already, anything that can use that as the engine/driver is good. Having looked at minikube, looked a bit scary to implement with "rootless" podman (?) so I plumped for `kind` instead.

## Deps

Kind requires go (v1.17+).

## Nice-to-haves

`kubectl` and `helm` are recommended for administering the cluster once it's up.

## steps

Record of what was done:

```
sudo pacman -S go helm kubectl # podman was already installed, but include it if in doubt
export PATH=$PATH:~/go/bin # maybe include in .bashrc so 'kind' will be in path when installed

go install sigs.k8s.io/kind@v0.29.0

# I needed to do this apparently, before it'll create the cluster
sudo modprobe ip6_tables

kind create cluster --name local
kubectl cluster-info --context kind-local # to confirm its working

helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard
helm install dashboard kubernetes-dashboard/kubernetes-dashboard -n kubernetes-dashboard --create-namespace
```

There is also a requirement to add a service account to authenticate dashboard access:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```bash
kubectl apply -f service-account.yaml

### Instructions show following, but is empty when I try it
# to fetch the token back - use the name/token in the UI
kubectl describe serviceaccount admin-user -n kubernetes-dashboard
kubectl describe secret admin-user-token-cn2fd -n kubernetes-dashboard

### My alternative is to create the token
kubectl -n kubernetes-dashboard create token admin-user # capture output

# Start the proxy and enable access to the web ui:
kubectl -n kubernetes-dashboard port-forward svc/dashboard-kong-proxy 8443:443 # https://localhost:8443
```
