# Lab 3
## 1. How many DaemonSets are created in the cluster in all namespaces?

One.

```console
$ kubectl get daemonsets --all-namespaces
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   24h
```

## 2. what DaemonSets exist on the kube-system namespace?

kube-proxy

## 3. What is the image used by the POD deployed by the kube-proxy DaemonSet

```console
$ kubectl describe daemonsets kube-proxy -n kube-system | grep -e [iI]mage
    Image:      registry.k8s.io/kube-proxy:v1.26.3
```

## 4. Deploy a DaemonSet for FluentD Logging. Use the given specifications

- Name: elasticsearch
- Namespace: kube-system
- Image: k8s.gcr.io/fluentd-elasticsearch:1.20

Daemonset definition: [fluentd.yml](./scripts/lab-3/fluentd.yml)

```console
$ kubectl apply -f scripts/lab-3/fluentd.yml &&
> sleep 5 && kubectl get daemonsets -n kube-system
daemonset.apps/elasticsearch created
NAME            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
elasticsearch   1         1         1       1            1           <none>                   5s
kube-proxy      1         1         1       1            1           kubernetes.io/os=linux   24h
```

## 5. Deploy a pod named nginx-pod using the nginx:alpine image with the labels set to tier=backend

Pod definition: [nginx-pod.yml](./scripts/lab-3/nginx-pod.yml)

```console
$ kubectl apply -f scripts/lab-3/nginx-pod.yml &&
> sleep 5 && kubectl get pods nginx-pod
pod/nginx-pod created
NAME        READY   STATUS              RESTARTS   AGE
nginx-pod   0/1     ContainerCreating   0          5s
```

## 6. Deploy a test pod using the nginx:alpine image

Deployment definition: [test.yml](./scripts/lab-3/test.yml)

```console
$ kubectl apply -f scripts/lab-3/test.yml &&
> sleep 5 && kubectl get pods test
pod/test created
NAME   READY   STATUS    RESTARTS   AGE
test   1/1     Running   0          5s
```

## 7. Create a service backend-service to expose the backend application within the cluster on port 80

Service definition: [backend-service.yml](./scripts/lab-3/backend-service.yml)

```console
$ kubectl apply -f scripts/lab-3/backend-service.yml &&
>  sleep 5 && kubectl get services backend-service
service/backend-service created
NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
backend-service   ClusterIP   10.100.20.201   <none>        80/TCP    5s
```

## 8. try to curl the backend-service from the test pod. What is the response?

```console
$ kubectl exec --stdin --tty test -- /bin/sh -c 'curl backend-service:80'
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 9. Create a deployment named web-app using the image nginx with 2 replicas

Deployment definition: [web-app.yml](./scripts/lab-3/web-app.yml)

```console
$ kubectl apply -f scripts/lab-3/web-app.yml &&
> sleep 5 && kubectl get deployments web-app
deployment.apps/web-app created
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   1/2     2            1           5s
$ kubectl get deployments web-app
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   2/2     2            2           9s
```

## 10. Expose the web-app as service web-app-service application on port 80 and nodeport 30082 on the nodes on the cluster

Service definition: [web-app-service.yml](./scripts/lab-3/web-app-service.yml)

```console
$ kubectl apply -f scripts/lab-3/web-app-service.yml &&
> sleep 5 && kubectl get services web-app-service
service/web-app-service created
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
web-app-service   NodePort   10.108.65.175   <none>        80:30082/TCP   5s
```

## 11. access the web app from the node

```console
$ kubectl get nodes minikube -o jsonpath=\
> '{.status.addresses[?(@.type=="InternalIP")].address}'
192.168.49.2
$ curl 192.168.49.2:30082
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## 12. How many static pods exist in this cluster in all namespaces?

Four.

```console
$ kubectl get pods -A -o custom-columns=\
> NAME:.metadata.name,\
> CONTROLLER:.metadata.ownerReferences[].kind,\
> NAMESPACE:.metadata.namespace,NODE:.spec.nodeName\
> | grep Node
etcd-minikube                      Node         kube-system   minikube
kube-apiserver-minikube            Node         kube-system   minikube
kube-controller-manager-minikube   Node         kube-system   minikube
kube-scheduler-minikube            Node         kube-system   minikube
```

## 13. On which nodes are the static pods created currently?

minikube
