# Kubernetes Workloads — Command Cheat Sheet

Quick reference for creating, testing, and cleaning up each workload type. Built and tested on a live 2-node K3s cluster.

---

## Pod

```
kubectl run nginx-pod --image=nginx --port=80
kubectl get pods
kubectl delete pod nginx-pod
```

---

## ReplicaSet

```yaml
# replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-rs
  template:
    metadata:
      labels:
        app: nginx-rs
    spec:
      containers:
      - name: nginx
        image: nginx
```

```
kubectl apply -f replicaset.yaml
kubectl get pods -l app=nginx-rs
kubectl delete pod <pod-name>          # test self-healing, a replacement appears
kubectl delete -f replicaset.yaml      # clean up
```

---

## Deployment

```
kubectl create deployment nginx-deploy --image=nginx --replicas=3
kubectl get deployments,replicasets,pods -l app=nginx-deploy
kubectl set image deployment/nginx-deploy nginx=nginx:1.27
kubectl rollout status deployment/nginx-deploy
kubectl get replicasets -l app=nginx-deploy   # see the new RS take over
kubectl delete deployment nginx-deploy
```

---

## DaemonSet

```yaml
# daemonset.yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nginx-daemonset
spec:
  selector:
    matchLabels:
      app: nginx-ds
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: nginx
        image: nginx
```

```
kubectl apply -f daemonset.yaml
kubectl get pods -l app=nginx-ds -o wide   # one Pod per node
kubectl delete -f daemonset.yaml
```

---

## StatefulSet

```yaml
# statefulset.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  clusterIP: None
  selector:
    app: nginx-sts
  ports:
  - port: 80
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: nginx-statefulset
spec:
  serviceName: nginx-headless
  replicas: 3
  selector:
    matchLabels:
      app: nginx-sts
  template:
    metadata:
      labels:
        app: nginx-sts
    spec:
      containers:
      - name: nginx
        image: nginx
```

```
kubectl apply -f statefulset.yaml
kubectl get pods -l app=nginx-sts      # -0, -1, -2, stable names
kubectl delete -f statefulset.yaml     # clean up when done
```

---

## Job

```yaml
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["sh", "-c", "echo Hello from a Job; sleep 5; echo Job finishing now"]
      restartPolicy: Never
```

```
kubectl apply -f job.yaml
kubectl get pods -l job-name=hello-job
kubectl logs -l job-name=hello-job
kubectl delete -f job.yaml
```

---

## CronJob

```yaml
# cronjob.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "* * * * *"
  jobTemplate:
    metadata:
      labels:
        app: hello-cronjob
    spec:
      template:
        metadata:
          labels:
            app: hello-cronjob
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["sh", "-c", "date; echo Hello from a scheduled Job"]
          restartPolicy: Never
```

```
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs -l app=hello-cronjob
kubectl logs -l app=hello-cronjob --tail=20
kubectl delete cronjob hello-cronjob    # cascades to its Jobs and Pods automatically
```

---

## One-time setup (if `kubectl` isn't already working)

```
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
export KUBECONFIG=~/.kube/config
echo 'export KUBECONFIG=~/.kube/config' >> ~/.bashrc
```
