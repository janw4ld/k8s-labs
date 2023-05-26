# Lab 2
## 1. How many Namespaces exist on the system?

```console
$ kubectl get ns --no-headers | wc -l
4
```

## 2. How many pods exist in the kube-system namespace?

```console
$ kubectl get po --no-headers -n kube-system | wc -l
7
```

## 3. Create a deployment with

- Name: beta
- Image: redis
- Replicas: 2
- Namespace: finance
- Resources Requests:
- CPU: .5 vcpu
- Mem: 1G
- Resources Limits:
- CPU: 1 vcpu
- Mem: 2G

Namespace definition: [finance.yml](./scripts/lab-2/finance.yml)
Deployment definition: [beta.yml](./scripts/lab-2/beta.yml)

```console
$ kubectl apply -f scripts/lab-2/finance.yml &&
> kubectl apply -f scripts/lab-2/beta.yml
namespace/finance created
deployment.apps/beta created
$ kubectl get deployments -n finance
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
beta   2/2     2            2           11s
```

## Multi Node Clusters

do the rest of the tasks on the following cluster
<https://www.katacoda.com/courses/kubernetes/playground>

> The site is no longer available so i've used <https://killercoda.com/playgrounds/scenario/kubernetes>.

### 4.How many Nodes exist on the system?  

Node count: 2

```console
$ kubectl get nodes
NAME           STATUS   ROLES           AGE     VERSION
controlplane   Ready    control-plane   5d15h   v1.27.1
node01         Ready    <none>          5d15h   v1.27.1
```

### 5. Do you see any taints on master?

No. Killercode removes controlplane taints by default.

```console
$ kubectl describe node controlplane | grep Taints
Taints:             <none>
```

or

```console
$ kubectl get nodes -o=jsonpath='{.items[].spec.taints}'
# no output
```

### 6. Apply a label color=blue to the master node

```console
$ kubectl label nodes controlplane color=blue && 
> kubectl get nodes -o=jsonpath='{.items[].metadata.labels}' \     
> | jq .
node/controlplane labeled
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "color": "blue",
  "kubernetes.io/arch": "amd64",
  "kubernetes.io/hostname": "controlplane",
  "kubernetes.io/os": "linux",
  "node-role.kubernetes.io/control-plane": "",
  "node.kubernetes.io/exclude-from-external-load-balancers": ""
}
```

<!-- I mistakenly tainted the node first -->
<!-- ```console
$ kubectl taint node controlplane color=blue:NoSchedule &&
> kubectl get nodes -o=jsonpath='{.items[].spec.taints}'
node/controlplane tainted
[{"effect":"NoSchedule","key":"color","value":"blue"}]
```
or

```console
$ kubectl taint node controlplane color=blue:NoSchedule
node/controlplane tainted
$ kubectl get nodes -o=jsonpath='{range .items[*]}{.metadata.name}{": "}{.spec.taints}{"\n"}{end}'
controlplane: [{"effect":"NoSchedule","key":"color","value":"blue"}]
node01:  
``` -->
### 7. Create a new deployment named blue with the nginx image and 3 replicas  
Set Node Affinity to the deployment to place the pods on master only

- NodeAffinity: requiredDuringSchedulingIgnoredDuringExecution
- key: color
- value: blue

Deployment definition: [blue-nginx.yml](./scripts/lab-2/blue-nginx.yml)

```console
$ kubectl apply -f blue-nginx.yml && 
> sleep 5 && kubectl get deployments
deployment.apps/blue created
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
blue   3/3     3            3           5s
```
