# Lab 5

## 1. Create a namespace haproxy-controller-devops

```console
$ kubectl create namespace haproxy-controller-devops
namespace/haproxy-controller-devops created
```

## 2. Create a ServiceAccount haproxy-service-account-devops under the same namespace

```console
$ kubectl create serviceaccount haproxy-service-account-devops \
> -n haproxy-controller-devops
serviceaccount/haproxy-service-account-devops created
```

## 3. Create a ClusterRole named haproxy-cluster-role-devops that

grants "get", "list", "watch", "create", "patch", "update" permissions to
"configmaps", "secrets", "endpoints", "nodes", "pods", "services", "events",
"namespaces", "serviceaccounts" resources.

ClusterRole definition: [haproxy-cluster-role-devops.yml](./scripts/lab-5/haproxy-cluster-role-devops.yml)

```console
$ kubectl apply -f scripts/lab-5/haproxy-cluster-role-devops.yml &&
> kubectl get clusterrole haproxy-cluster-role-devops
clusterrole.rbac.authorization.k8s.io/haproxy-cluster-role-devops created
NAME                          CREATED AT
haproxy-cluster-role-devops   2023-05-27T15:23:20Z
```

## 4. Create a ClusterRoleBinding according to the following spec

    Name: haproxy-cluster-role-binding-devops
    Namespace: haproxy-controller-devops
    Role Reference:
      Name: haproxy-cluster-role-devops
      Kind: ClusterRole
      apiGroup: rbac.authorization.k8s.io
    Subjects:
      Name: haproxy-service-account-devops
      Kind: ServiceAccount
      Namespace: haproxy-controller-devops

ClusterRoleBinding definition: [haproxy-cluster-role-binding-devops.yml](./scripts/lab-5/haproxy-cluster-role-binding-devops.yml)

```console
$ kubectl apply -f scripts/lab-5/haproxy-cluster-role-binding-devops.yml && 
> kubectl get clusterrolebinding -n haproxy-controller-devops \
> haproxy-cluster-role-binding-devops
clusterrolebinding.rbac.authorization.k8s.io/haproxy-cluster-role-binding-devops created
NAME                                  ROLE                                      AGE
haproxy-cluster-role-binding-devops   ClusterRole/haproxy-cluster-role-devops   0s
```

## 5. Create a backend deployment under the namespace "haproxy-controller-devops" according to the following spec

    Name: backend-deployment-devops
    Labels:
      run: ingress-default-backend
    Spec:
      Replica: 1
      Selector labels:
        run: ingress-default-backend
      Template labels:
        run: ingress-default-backend
      Container:
        Name: backend-container-devops
        Image: gcr.io/google_containers/defaultbackend:1.0 (use the exact image name as specified)
        ContainerPort: 8080

Deployment definition: [backend-deployment-devops.yml](./scripts/lab-5/backend-deployment-devops.yml)

```console
$ kubectl apply -f scripts/lab-5/backend-deployment-devops.yml &&
> sleep 5 && kubectl get deployment -n haproxy-controller-devops \
> backend-deployment-devops
deployment.apps/backend-deployment-devops created
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
backend-deployment-devops   1/1     1            1           5s
```

## 6. Create a service for the backend named "service-backend-devops" under the same namespace according to the following spec

    Labels:
      run: ingress-default-backend
    Spec:
      Selector labels:
        run: ingress-default-backend
      Port:
        Name: port-backend
        Protocol: TCP
        Port: 8080
        TargetPort: 8080

Service definition: [service-backend-devops.yml](./scripts/lab-5/service-backend-devops.yml)
  
```console
$ kubectl apply -f scripts/lab-5/service-backend-devops.yml &&
> kubectl get service -n haproxy-controller-devops service-backend-devops
service/service-backend-devops created
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service-backend-devops   ClusterIP   10.102.88.83   <none>        8080/TCP   0s
```

## 7. Create a deployment for the frontend named "haproxy-ingress-devops" under the same namespace according to the following spec

    Spec:
      Replica: 1
      Selector labels:
        haproxy-ingress
      Template labels:
        run: haproxy-ingress
      Containers:
        Name: ingress-container-devops
        Image: haproxytech/kubernetes-ingress
        Args:
          --default-backend-service=haproxy-controller-devops/service-backend-devops
        Resources:
          Requests:
            CPU: 500m
            Memory: 50Mi
        Liveness Probe:
          HTTPGet:
            Path: /healthz
            Port: 1024
      Ports:
        Name: http
          ContainerPort: 80
        Name: https
          ContainerPort: 443
        Name: stat
          ContainerPort: 1024
      Env:
        Name: TZ
          Value: Etc/UTC
        Name: POD_NAME
          ValueFrom:
            FieldRef:
              FieldPath: metadata.name
        Name: POD_NAMESPACE
          ValueFrom:
            FieldRef:
              FieldPath: metadata.namespace

Deployment definition: [haproxy-ingress-devops.yml](./scripts/lab-5/haproxy-ingress-devops.yml)

```console
$ kubectl apply -f scripts/lab-5/haproxy-ingress-devops.yml &&
> sleep 5 && kubectl get deployment -n haproxy-controller-devops \
> haproxy-ingress-devops
deployment.apps/haproxy-ingress-devops created
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
haproxy-ingress-devops   0/1     1            0           5s
$ kubectl get deployment -n haproxy-controller-devops haproxy-ingress-devops
NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
haproxy-ingress-devops   1/1     1            1           39s
```

## 8. Create a service for the frontend named "ingress-service-devops" under the same namespace according to the following spec

    Labels:
      run: haproxy-ingress
    Spec:
      Selector labels:
        run: haproxy-ingress
      Type: NodePort
      Ports:
        Name: http
          Protocol: TCP
          Port: 80
          TargetPort: 80
          NodePort: 32456
        Name: https
          Protocol: TCP
          Port: 443
          TargetPort: 443
          NodePort: 32567
        Name: stat
          Protocol: TCP
          Port: 1024
          TargetPort: 1024
          NodePort: 32678

Service definition: [ingress-service-devops.yml](./scripts/lab-5/ingress-service-devops.yml)
  
```console
$ kubectl apply -f scripts/lab-5/ingress-service-devops.yml &&
> kubectl get service -n haproxy-controller-devops ingress-service-devops
service/ingress-service-devops created
NAME                     TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                                     AGE
ingress-service-devops   NodePort   10.107.254.241   <none>        80:32456/TCP,443:32567/TCP,1024:32678/TCP   0s
```
