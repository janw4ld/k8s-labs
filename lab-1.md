# Lab 1

## 1. Install k8s cluster (minikube)

```bash
set -e # exit on any error

# install minikube on fedora
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
sudo rpm -Uvh minikube-latest.x86_64.rpm

# install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# configure minikube to use docker driver
minikube config set driver docker

# install bash completion for minikube and kubectl
echo "source <(minikube completion bash)" >> ~/.bashrc
echo "source <(kubectl completion bash)" >> ~/.bashrc

# start minikube
minikube start
```

## 2. Create a pod with the name redis and with the image redis.

```console
$ kubectl run redis --image=redis                       
pod/redis created                      
```

## 3. Create a pod with the name nginx and with the image nginx123
Use a pod-definition YAML file.

Pod definition: [nginx-pod.yml](./scripts/lab-1/nginx-pod.yml)

Execution and status check:

```bash
$ kubectl apply -f scripts/lab-1/nginx-pod.yml &&
> sleep 5 && kubectl get pods nginx-pod
pod/nginx-pod created
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          5s
```

## 4. What is the nginx pod status?
Running

## 5. Change the nginx pod image to “nginx” & check the status again

Pod definition: [nginx-pod_2.yml](./scripts/lab-1/nginx-pod_2.yml)

Execution and status check:

```bash
$ kubectl apply -f scripts/lab-1/nginx-pod_2.yml &&
> sleep 5 && kubectl get pods nginx-pod
pod/nginx-pod configured
NAME        READY   STATUS    RESTARTS     AGE
nginx-pod   1/1     Running   1 (5s ago)   3m14s
```

## 6. How many ReplicaSets exist on the system?
Zero.

```console
$ kubectl get replicasets --no-headers | wc -l
No resources found in default namespace. # stderr
0                                        # stdout line count
```

## 7. create a ReplicaSet with

- name= replica-set-1
- image= busybox
- replicas= 3

ReplicaSet definition: [rs-busybox.yml](./scripts/lab-1/rs-busybox.yml)

Execution and status check:

```console
$ kubectl apply -f scripts/lab-1/rs-busybox.yml &&
> sleep 5 && kubectl get rs
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   3         3         2       5s
$ kubectl get rs
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   3         3         3       6s
```

## 8. Scale the ReplicaSet replica-set-1 to 5 PODs

ReplicaSet definition: [rs-busybox_2.yml](./scripts/lab-1/rs-busybox_2.yml)

Execution and status check:

```console
$ kubectl apply -f scripts/lab-1/rs-busybox_2.yml &&
> sleep 5 && kubectl get rs
replicaset.apps/replica-set-1 configured
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   5         5         5       1m37s
```

## 9. How many PODs are READY in the replica-set-1?
Five.

## 10. Delete any one of the 5 PODs then check How many PODs exist now?
Five.

```console
$ kubectl get pods
NAME                  READY   STATUS    RESTARTS      AGE
nginx-pod             1/1     Running   3 (27m ago)   34m
redis                 1/1     Running   0             57m
replica-set-1-8rd72   1/1     Running   0             6m21s
replica-set-1-hns58   1/1     Running   0             6m21s
replica-set-1-qcbb9   1/1     Running   0             2m49s
replica-set-1-w7sw7   1/1     Running   0             2m49s
replica-set-1-wglp5   1/1     Running   0             64s
$ kubectl delete po replica-set-1-8rd72 && kubectl get rs
pod "replica-set-1-8rd72" deleted
NAME            DESIRED   CURRENT   READY   AGE
replica-set-1   5         5         5       7m3s
```

### Why are there still 5 PODs, even after you deleted one?
Because the ReplicaSet's job is to ensure that the desired number of Pods is running, so if a pod is deleted, the ReplicaSet will create a new one to replace it to keep the running pods at 5.

## 11. How many Deployments and ReplicaSets exist on the system?
Zero deployments and one replicaset.

```console
$ kubectl get deployments --no-headers | wc -l
No resources found in default namespace.
0
$ kubectl get rs --no-headers | wc -l
1
```

or, with one command

```console
$ kubectl get deployments,rs
NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/replica-set-1   5         5         5       12m
```

## 12. create a Deployment with

- name= deployment-1
- image= busybox
- replicas= 3

Deployment definition: [deployment-1.yml](./scripts/lab-1/deployment-1.yml)

Execution and status check:

```console
$ kubectl apply -f scripts/lab-1/deployment-1.yml &&
> sleep 5 && kubectl get deployments
deployment.apps/deployment-1 created
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   2/3     3            2           6s
$ kubectl get deployments
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   3/3     3            3           11s
```

## 13. How many Deployments and ReplicaSets exist on the system now?
One deployment and two replicasets.

```console
$ kubectl get deployments,rs
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/deployment-1   3/3     3            3           2m4s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/deployment-1-65fd7d9646   3         3         3       2m4s
replicaset.apps/replica-set-1             5         5         5       19m
```

## 14. How many pods are ready with the deployment-1?
Three.

```console
$ kubectl get deployments deployment-1 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   3/3     3            3           2m51s
```

## 15. Update deployment-1 image to nginx then check the ready pods again
Deployment definition: [deployment-1_2.yml](./scripts/lab-1/deployment-1_2.yml)

Execution and status check:

```console
$ kubectl apply -f scripts/lab-1/deployment-1_2.yml &&
> sleep 5 && kubectl get deployments
deployment.apps/deployment-1 configured
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1   3/3     3            3           9m26s
```

## 16. Run kubectl describe deployment deployment-1 and check events
What is the deployment strategy used to upgrade the deployment-1?  
`StrategyType:  RollingUpdate`

## 17. Rollback the deployment-1
What is the used image with the deployment-1?
busybox

```console
$ kubectl rollout undo deployment/deployment-1 &&
> kubectl describe deployment deployment-1 | grep Image:
    Image:      busybox
```

## 18. Create a deployment using nginx image with latest tag only and remember to mention tag i.e nginx:latest and name it as nginx-deployment. App labels should be app: nginx-app and type: front-end. The container should be named as nginx-container; also make sure replica count is 3

Deployment definition: [nginx-deployment.yml](./scripts/lab-1/nginx-deployment.yml)

Execution and status check:

```console
$ kubectl apply -f scripts/lab-1/nginx-deployment.yml &&
> sleep 5 && kubectl get deployments
deployment.apps/nginx-deployment created
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1       3/3     3            3           20m
nginx-deployment   2/3     3            2           5s
$ kubectl get deployments
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
deployment-1       3/3     3            3           20m
nginx-deployment   3/3     3            3           11s
```
