# Lab 4

## 1. How many ConfigMaps exist in the environment?

Ten.

```console
$ kubectl get configmaps --all-namespaces --no-headers \
> | tee >(wc -l)  # print to stdout and wc -l to count
default           kube-root-ca.crt                     1     46h
kube-node-lease   kube-root-ca.crt                     1     46h
kube-public       cluster-info                         1     46h
kube-public       kube-root-ca.crt                     1     46h
kube-system       coredns                              1     46h
kube-system       extension-apiserver-authentication   6     46h
kube-system       kube-proxy                           2     46h
kube-system       kube-root-ca.crt                     1     46h
kube-system       kubeadm-config                       1     46h
kube-system       kubelet-config                       1     46h
10
```

## 2. Create a new ConfigMap  Use the spec given below

- ConfigName Name: webapp-config-map
- Data: APP_COLOR=darkblue

ConfigMap definition: [web-app-config-map.yml](./scripts/lab-4/web-app-config-map.yml)

```console
$ kubectl apply -f scripts/lab-4/webapp-config-map.yml &&
> kubectl get configmaps webapp-config-map
configmap/webapp-config-map created
NAME                DATA   AGE
webapp-config-map   1      0s
```

## 3. Create a webapp-color POD with nginx image and use the previously created ConfigMap

Pod definition: [webapp-color.yml](./scripts/lab-4/webapp-color.yml)

```console
$ kubectl apply -f scripts/lab-4/webapp-color.yml &&
> sleep 5 && kubectl get pods webapp-color

pod/webapp-color created
NAME           READY   STATUS    RESTARTS   AGE
webapp-color   1/1     Running   0          5s
$ kubectl exec --stdin --tty webapp-color -- /bin/sh -c 'echo $APP_COLOR'
darkblue
```

## 4. How many Secrets exist on the system?

Zero.

```console
$ kubectl get secrets --all-namespaces --no-headers | tee >(wc -l)
No resources found
0
```

## 5. How many secrets are defined in the default-token secret?

The secret is not defined in the current minikube=v1.30.1 environment.

## 6. create a POD called db-pod with the image mysql:5.7 then check the POD status

```console
$ kubectl run db-pod --image=mysql:5.7 && sleep 5 && kubectl get po db-pod
pod/db-pod created
NAME     READY   STATUS             RESTARTS     AGE
db-pod   0/1     CrashLoopBackOff   1 (3s ago)   5s
```

## 7. why the db-pod status not ready

MySQL server crashes because it requires credentials to be set.
  
```console
$ kubectl logs db-pod --previous
2023-05-27 10:38:24+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL
Server 5.7.42-1.el7 started.
2023-05-27 10:38:24+00:00 [Note] [Entrypoint]: Switching to dedicated user
'mysql'
2023-05-27 10:38:24+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL
Server 5.7.42-1.el7 started.
2023-05-27 10:38:24+00:00 [ERROR] [Entrypoint]: Database is uninitialized and
password option is not specified
    You need to specify one of the following as an environment variable:
    - MYSQL_ROOT_PASSWORD
    - MYSQL_ALLOW_EMPTY_PASSWORD
    - MYSQL_RANDOM_ROOT_PASSWORD
```

## 8. Create a new secret named db-secret with the data given below

    Secret Name: db-secret
      Secret 1: MYSQL_DATABASE=sql01
      Secret 2: MYSQL_USER=user1
      Secret 3: MYSQL_PASSWORD=password
      Secret 4: MYSQL_ROOT_PASSWORD=password123

```console
$ cat<<EOF >scripts/lab-4/db-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  MYSQL_DATABASE: $(echo -n sql01 | base64)
  MYSQL_USER: $(echo -n user1 | base64)
  MYSQL_PASSWORD: $(echo -n password | base64)
  MYSQL_ROOT_PASSWORD: $(echo -n password123 | base64)
EOF
$ kubectl apply -f scripts/lab-4/db-secret.yml \
> && kubectl get secrets db-secret
secret/db-secret created
NAME        TYPE     DATA   AGE
db-secret   Opaque   4      0s
```

## 9. Configure* db-pod to load environment variables from the newly created secret

<sup>* Delete and recreate the pod if required.</sup>

Pod definition: [db-pod_2.yml](./scripts/lab-4/db-pod_2.yml)

```console
$ kubectl apply -f scripts/lab-4/db-pod_2.yml --force \
> && sleep 5 && kubectl get po db-pod
Warning: resource pods/db-pod is missing the
kubectl.kubernetes.io/last-applied-configuration annotation which is required by
kubectl apply. kubectl apply should only be used on resources created
declaratively by either kubectl create --save-config or kubectl apply. The
missing annotation will be patched automatically.
pod/db-pod configured
NAME     READY   STATUS    RESTARTS   AGE
db-pod   1/1     Running   0          5s  # automatically deleted by --force
```

## 10. Create a multi-container pod with 2 containers

    Name: yellow
      Container 1 Name: lemon
      Container 1 Image: busybox
      Container 2 Name: gold
      Container 2 Image: redis

Pod definition: [yellow.yml](./scripts/lab-4/yellow.yml)

```console
$ kubectl apply -f scripts/lab-4/yellow.yml \
> && sleep 5 && kubectl get po yellow
pod/yellow created
NAME     READY   STATUS     RESTARTS   AGE
yellow   1/2     NotReady   0          5s
$ kubectl logs yellow --all-containers
1:C 27 May 2023 11:13:22.409 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
1:C 27 May 2023 11:13:22.409 # Redis version=7.0.11, bits=64, commit=00000000, modified=0, pid=1, just started
1:C 27 May 2023 11:13:22.409 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
1:M 27 May 2023 11:13:22.409 * monotonic clock: POSIX clock_gettime
1:M 27 May 2023 11:13:22.409 * Running mode=standalone, port=6379.
1:M 27 May 2023 11:13:22.409 # Server initialized
1:M 27 May 2023 11:13:22.410 * Ready to accept connections
```

gold (busybox) is crashing because it requires a command.

## 11. Create a pod red with redis image and use an initContainer that uses the busybox image and sleeps for 20 seconds

Pod definition: [red.yml](./scripts/lab-4/red.yml)

```console
$ kubectl apply -f scripts/lab-4/red.yml && { 
>   (sleep 5 && kubectl get po red) &&
>   (sleep 20 && kubectl get po red)
> }
pod/red created
NAME   READY   STATUS     RESTARTS   AGE
red    0/1     Init:0/1   0          5s
NAME   READY   STATUS    RESTARTS   AGE
red    1/1     Running   0          25s
```

## 12. Create a pod named print-envars-greeting

1. Configure spec as, the container name should be
print-env-container and use bash image.
1. Create three environment variables:
    - GREETING and its value should be “Welcome to”
    - COMPANY and its value should be “DevOps”
    - GROUP and its value should be “Industries”
1. Use command to echo ["$(GREETING) $(COMPANY) $(GROUP)"]
message.

Pod definition: [print-envars-greeting.yml](./scripts/lab-4/print-envars-greeting.yml)

```console
$ kubectl apply -f scripts/lab-4/print-envars-greeting.yml &&
> sleep 5 && kubectl logs print-envars-greeting
pod/print-envars-greeting created
Welcome to DevOps Industries
```

## 13. Where is the default kubeconfig file located in the current environment?

Most likely in `$HOME/.kube/config`.

```console
$ kubectl config view --v=6 | grep "Config loaded"
I0527 14:38:39.947356 3668083 loader.go:373] Config loaded from file:  /home/meka/.kube/config
```

## 14. How many clusters are defined in the default kubeconfig file?

One.

```console
$ kubectl config view -o jsonpath='{.clusters[].name}{"\n"}' \
> | tee >(wc -l)
minikube
1
```

## 15. What is the user configured in the current context?

```console
$ kubectl config view -o jsonpath=\
> '{.contexts[?(@.name == "'$(kubectl config current-context)'")].context.user}'
minikube
```

## 16. Create a Persistent Volume with the given specification.

    Volume Name: pv-log
      Storage: 100Mi
      Access Modes: ReadWriteMany
      Host Path: /pv/log

PV definition: [pv-log.yml](./scripts/lab-4/pv-log.yml)

```console
$ kubectl apply -f scripts/lab-4/pv-log.yml
persistentvolumes/pv-log created
```

## 17. Create a Persistent Volume Claim with the given specification.

    Volume Name: claim-log-1
      Storage Request: 50Mi
      Access Modes: ReadWriteMany

PVC definition: [claim-log-1.yml](./scripts/lab-4/claim-log-1.yml)

```console
$ kubectl apply -f scripts/lab-4/claim-log-1.yml
persistentvolumeclaims/claim-log-1 created
```

## 18. Create a webapp pod to use the persistent volume claim as its storage

    Name: webapp
      Image Name: nginx
      Volume: PersistentVolumeClaim=claim-log-1
      Volume Mount: /var/log/nginx

Pod definition: [webapp.yml](./scripts/lab-4/webapp.yml)
  
```console
$ kubectl apply -f scripts/lab-4/webapp.yml \
> && sleep 5 && kubectl get po webapp
pod/webapp created
NAME     READY   STATUS    RESTARTS   AGE
webapp   1/1     Running   0          5s
```
