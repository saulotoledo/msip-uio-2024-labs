# Kubernetes Lab

- [Kubernetes Lab](#kubernetes-lab)
  - [Before starting](#before-starting)
  - [Task K1: Playing with a local Kubernetes cluster using Kind:](#task-k1-playing-with-a-local-kubernetes-cluster-using-kind)
  - [Task K2: Our first pod](#task-k2-our-first-pod)
  - [Task K3: Our first ReplicaSet](#task-k3-our-first-replicaset)
  - [Task K4: Using a new version of the container](#task-k4-using-a-new-version-of-the-container)
  - [Task K5: Deployments](#task-k5-deployments)
  - [Task K6: Creating a Service](#task-k6-creating-a-service)


## Before starting

Check the instructions on the [previous page](README.md).

In addition to that, we need the following software installed:

- **Kind:** [https://kind.sigs.k8s.io/docs/user/quick-start/#installation](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- **Kubectl** (CLI for the cluster): [https://kubernetes.io/docs/tasks/tools/](https://kubernetes.io/docs/tasks/tools/)

## Task K1: Playing with a local Kubernetes cluster using Kind:

Run the following commands one by one in order and check their outputs:

```sh
kind create cluster --name tempcluster
kind get clusters
kubectl cluster-info
kind delete cluster -n tempcluster
kind create cluster --name ourcatalog
kubectl get nodes
kubectl cluster-info
docker container ls
```

## Task K2: Our first pod

Create a folder named `k8s-tutorial` and change to it:

```sh
mkdir k8s-tutorial
cd k8s-tutorial
```

Create a file named `pod.yml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: msip-products
  labels:
    type: products
spec:
   containers:
     - name: msip-products-container
       image: saulotoledo/ourcatalog-products:v1
       env:
         - name: PORT
           value: "3000"
         - name: HOST
           value: "0.0.0.0"
         - name: NODE_ENV
           value: "production"
         - name: APP_KEY
           value: "adonisjs-appkey-change-me"
         - name: PAGINATION_LIMIT
           value: "10"
         - name: DB_CONNECTION
           value: "sqlite3"
```

Apply the manifest and list your pods:

```sh
kubectl apply -f pod.yml
kubectl get pods
kubectl get po
```

You need to expose the Pod to access it locally using the `port-forward` command:

```sh
kubectl port-forward pod/msip-products 8080:3000
```

Finnaly, open [http://localhost:8080/api/products](http://localhost:8080/api/products) in your browser.


## Task K3: Our first ReplicaSet

Stop the `port-forwarding` from the previous step with `Ctrl + C` and remove the pod we created before:

```sh
kubectl delete pod msip-products
```

Create a file named `replicaset.yml` with the following content:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: msip-products
spec:
  replicas: 3
  selector:
    matchLabels:
      app: products
  template:
    metadata:
      labels:
        app: products
    spec:
      containers:
        - name: msip-products-container
          image: saulotoledo/ourcatalog-products:v1
          env:
            - name: PORT
              value: "3000"
            - name: HOST
              value: "0.0.0.0"
            - name: NODE_ENV
              value: "production"
            - name: APP_KEY
              value: "adonisjs-appkey-change-me"
            - name: PAGINATION_LIMIT
              value: "10"
            - name: DB_CONNECTION
              value: "sqlite3"
```

Apply the manifest and check ReplicaSets and Pods:

```sh
kubectl apply -f replicaset.yml
kubectl get replicasets
kubectl get po
```

Remove one Pod created by the ReplicaSet, then recheck the Pods:

```sh
kubectl delete po (PodName)
kubectl get po
```

Note that a new Pod was created to restore the previous state.

Finally, increase replicas and reapply the manifest. Note that replicas will change to match the value you changed.


## Task K4: Using a new version of the container

Version 2 of our "products" service has a new endpoint `/info`. Let's release it in our cluster!

Start by checking our pods:

```sh
kubectl get po

NAME                  READY   STATUS    RESTARTS   AGE
msip-products-8fn6n   1/1     Running   0          9m6s
msip-products-9phgb   1/1     Running   0          8m18s
msip-products-pjlhd   1/1     Running   0          9m6s
```

Next, open our `replicaset.yml` file and change `:v1` with `:v2` in the container tag. Reapply the manifest and check our pods again:

```sh
kubectl apply -f replicaset.yml
kubectl get po

NAME                  READY   STATUS    RESTARTS   AGE
msip-products-8fn6n   1/1     Running   0          11m
msip-products-9phgb   1/1     Running   0          10m
msip-products-pjlhd   1/1     Running   0          11m
```

Something seems wrong; the pods did not change even if the ReplicaSet was applied successfully.

Describe one of the pods:

```sh
kubectl describe pod (PodName)
```

And... wait! The Pod still uses the version `v1`! Let's delete this Pod.

```sh
kubectl delete pod (PodName)
```

Recheck the Pods: the ReplicaSet will restore the missing one. Describe this new pod and see that it uses `v2` now:

```sh
kubectl describe pod (NewPodName)
```

But the other Pods still use `v1` until you remove them. How can we fix this?


## Task K5: Deployments

Remove the previous ReplicaSet:

```sh
kubectl delete replicaset msip-products
```

Create a copy of the `replicaset.yml` as `deployment.yml`.

Restore the version of the image to `v1` and replace the value of `kind` with `Deployment`. Next, apply the manifest:

```sh
kubectl apply -f deployment.yml
```

Change the version of the image to `v2` and reapply the manifest. Then, check the pods a few times. Note that the Deployment will terminate the old pods and create new ones in place:

```sh
kubectl apply -f deployment.yml
kubectl get po
```

Finally, let's scale the Deployment:

```sh
kubectl scale deployment msip-products --replicas=10
```

## Task K6: Creating a Service

A Service helps us access the application and may act as a load balancer. Create a file named `service.yml` with the following content:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: msip-products
spec:
  # Filter pods to use (see matchLabels in Deployment)
  selector:
    app: products
  type: ClusterIP
  ports:
    - name: msip-products-port
      port: 80 # Service port
      targetPort: 3000 # Container port
      protocol: TCP
```

Apply the manifest and check the services. Note that it has an internal IP:

```sh
kubectl apply -f service.yml
kubectl get services
```

Finally, use the `port-forward` command to expose the service to local access:

```sh
kubectl port-forward svc/msip-products 8080:80
```
